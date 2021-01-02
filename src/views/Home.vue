<template>
  <div class="home">
    <el-row type="flex" class="row-bg" justify="center">
      <el-col :sm="24" :md="16">
        <div class="grid-content bg-purple">
          <el-card :body-style="{ padding: '20px' }">
            <el-row>
              <el-col :span="12">
                <h3>Dane samochodu</h3>
                <el-form label-position="left" label-width="100px" :model="carModel">
                  <el-form-item label="Marka">
                    <el-input v-model="carModel.mark"></el-input>
                  </el-form-item>
                  <el-form-item label="Model">
                    <el-input v-model="carModel.model"></el-input>
                  </el-form-item>
                  <el-form-item label="Rejestracja">
                    <el-input v-model="carModel.plate"></el-input>
                  </el-form-item>
                </el-form>
              </el-col>
            </el-row>

            <el-row :gutter="20">
              <el-col :sm="24" :md="12">
                <h3>Formularz</h3>
                <el-form label-position="left" label-width="100px" :model="formModel">
                  <el-form-item label="Rok">
                    <el-select v-model="yearModel" placeholder="wybierz" @change="clearDay">
                      <el-option v-for="item in years" :key="item" :label="item" :value="item"> </el-option>
                    </el-select>
                  </el-form-item>
                  <el-form-item label="Miesiąc">
                    <el-select v-model="monthModel" placeholder="wybierz" @change="onChange">
                      <el-option v-for="item in options" :key="item.id" :label="item.value" :value="item"> </el-option>
                    </el-select>
                  </el-form-item>
                  <el-form-item label="Dzień">
                    <el-select v-model="dayModel" placeholder="wybierz">
                      <el-option v-for="item in days" :key="item.value" :label="item.value" :value="item"> </el-option>
                    </el-select>
                  </el-form-item>
                  <el-form-item label="Kilometry">
                    <el-input type="number" v-model="formModel.km"></el-input>
                  </el-form-item>
                  <el-form-item label="Adres">
                    <el-input v-model="formModel.addressTo"></el-input>
                  </el-form-item>
                  <div class="form-buttons">
                    <el-button @click="clearForm">
                      Wyczyść
                    </el-button>
                    <el-button type="success" @click="addItem">Dodaj dzień</el-button>
                  </div>
                </el-form>
              </el-col>
              <el-col :sm="24" :md="12">
                <h3>Podsumowanie</h3>
                <el-form label-position="left" label-width="100px">
                  <el-form-item label="Kilometry" class="align-left">
                    <span>{{ getAllKm }}</span>
                  </el-form-item>
                  <el-form-item label="Stawka">
                    <el-input type="number" v-model="rate"></el-input>
                  </el-form-item>
                  <el-form-item label="Kwota" class="align-left">
                    <span>{{ getResult }}</span>
                  </el-form-item>
                  <el-form-item label="Nazwa pliku">
                    <el-input v-model="fileName"></el-input>
                  </el-form-item>
                  <div class="form-buttons">
                    <el-button :disabled="!tableData.length" type="primary" @click="downoloadPdf">Pobierz PDF</el-button>
                  </div>
                </el-form>
              </el-col>
            </el-row>
          </el-card>
        </div>
      </el-col>
    </el-row>
    <el-row type="flex" justify="center" class="table-container">
      <el-col :sm="24" :md="16">
        <div>
          <el-table :data="tableData" empty-text="Brak danych" style="width: 100%">
            <el-table-column prop="week" label="Nr tyg" width="180"> </el-table-column>
            <el-table-column prop="date" label="Data" width="180"> </el-table-column>
            <el-table-column prop="km" label="Kilometry" width="180"> </el-table-column>
            <el-table-column prop="addressFrom" label="Twój adres"> </el-table-column>
            <el-table-column prop="addressTo" label="Adres wyjazdu"> </el-table-column>
            <el-table-column fixed="right" label="Akcje" width="120">
              <template v-slot="{ $index }">
                <el-button type="text" size="small" @click="removeRow($index)">Usuń</el-button>
              </template>
            </el-table-column>
          </el-table>
        </div>
      </el-col>
    </el-row>
    <div id="pdfHtml" style="display: none">
      <p>Bedrijfsnaam KW Super Schoon</p>
      <p>Adres Valkhof 159</p>
      <p>Postcode & plaats 3362GG Sliedrecht</p>
    </div>
    <div id="editor" style="display: none;">
      <p>Merk auto: {{ carModel.mark }}</p>
      <p>Type autoo: {{ carModel.model }}</p>
      <p>Kenteken auto: {{ carModel.plate }}</p>
    </div>
  </div>
</template>

<script lang="ts">
/* eslint-disable */
import { Component, Vue } from 'vue-property-decorator';
import moment from 'moment';
import jsPDF from 'jspdf';
import 'jspdf-autotable';
moment.locale('nl');

@Component({
  components: {},
})
export default class Home extends Vue {
  ADDRESS: string = '3362GG Sliedrecht';
  EURO_SIGN: string = '\u20ac';
  PDF_EXTENSION: string = '.pdf';
  PDF_HEAD: Array<Array<string>> = [['Weeknummer', 'Datum', 'Kilometers', 'Vertrekadres', 'Aankomstadres']];
  formModel: any = { date: '', km: '', week: '', addressFrom: this.ADDRESS, addressTo: '' };
  days: Array<any> = [];
  monthModel: string = '';
  options: Array<Object> = moment.months().map((date, index) => ({ id: index, value: this.capitalize(date) }));
  data: Array<any> = [];
  tableData: Array<any> = [];
  dayModel: any = '';
  rate: number = 0.19;
  result: number = 0;
  chosenMonth: string = '';
  fileName: string = 'plik';
  carModel: any = { mark: 'Volkswagen', model: 'Polo', plate: '49-PJ-VS' };
  yearModel: number = new Date().getFullYear();
  years: number[] = [2019, 2020, 2021, 2022, 2023, 2024, 2025];

  get getDays() {
    return this.days;
  }

  get getFormValidation() {
    return Object.values(this.formModel).every((val) => !!val);
  }

  get getAllKm(): any {
    let sum = 0;
    this.tableData.map((el) => {
      sum += parseInt(el.km);
    });
    return sum.toFixed(2);
  }

  get getResult() {
    return this.EURO_SIGN + (this.getAllKm * this.rate).toFixed(2);
  }

  onChange(item: any) {
    this.chosenMonth = item.value;
    this.createDays(item.id);
  }

  createDays(month: number): any {
    this.days = [];
    let date = moment(new Date(this.yearModel, month));
    let daysCount = date.daysInMonth();
    let days: any = {};
    let startIndex = date.weekday();
    for (let i = 1; i <= daysCount; i++) {
      let dateObj = moment(new Date(this.yearModel, month, i));
      let weekNumber = dateObj.week();
      let stringDate = dateObj.format('MMMM DD,YYYY');
      let stringDay = moment.weekdays(startIndex + i);
      if (!days[weekNumber]) days[weekNumber] = [];
      days[weekNumber].push(`${stringDay}, ${stringDate}`);
      this.days.push({ value: `${this.capitalize(stringDay)}, ${stringDate}`, week: weekNumber });
    }
  }

  clearForm() {
    this.formModel = { addressFrom: '3362GG Sliedrecht' };
    this.clearDay();
    this.yearModel = new Date().getFullYear();
  }

  clearDay() {
    this.dayModel = {};
    this.monthModel = '';
  }

  addItem() {
    this.formModel.date = this.dayModel.value;
    this.formModel.week = this.dayModel.week;
    if (this.getFormValidation) {
      this.tableData.push({ ...this.formModel });
      this.clearForm();
    }
  }

  createPdfContent() {
    this.data = [];
    this.days.map((day) => {
      const list = this.tableData.filter((el) => el.date === day.value);
      if (list.length) {
        list.map((item) => {
          let arr = [];
          arr.push(day.week);
          arr.push(item.date);
          arr.push(item.km);
          arr.push(item.addressFrom);
          arr.push(item.addressTo);
          this.data.push([...arr]);
        });
      } else {
        let arr = [];
        arr.push(day.week);
        arr.push(day.value);
        arr.push('');
        arr.push(this.ADDRESS);
        arr.push('');
        this.data.push([...arr]);
      }
    });
  }

  capitalize(value: string) {
    return value.charAt(0).toUpperCase() + value.slice(1);
  }

  removeRow(index: number) {
    this.tableData.splice(index, 1);
  }

  downoloadPdf() {
    this.createPdfContent();
    const doc: any = new jsPDF();

    let id = document.querySelector('#pdfHtml');
    let editor = document.querySelector('#editor');
    doc.fromHTML(id, 14, 10, {});
    doc.text(`Periode: ${this.chosenMonth} ${moment().year()}`, 150, 10);
    doc.fromHTML(editor, 14, 40, { bottom: 50 });
    doc.autoTable({ showHead: 'firstPage', startY: 80, head: this.PDF_HEAD, body: this.data });
    doc.autoTable({ head: [['Kilometers', 'Tarief', 'Bedrag']], body: [[this.getAllKm, this.EURO_SIGN + this.rate, this.getResult]] });
    doc.save(this.fileName + this.PDF_EXTENSION);
  }
}
</script>
<style lang="scss" scoped>
.home {
  width: 100%;
}

.el-select {
  width: 100%;
}

.align-left {
  text-align: left;
}

.form-buttons {
  display: flex;
  justify-content: flex-end;
}

.table-container {
  margin: 30px 0;
}
</style>
