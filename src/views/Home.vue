<template>
  <div class="home">
    <el-row
      type="flex"
      class="row-bg"
      justify="center"
    >
      <el-col :span="16">
        <div class="grid-content bg-purple">
          <el-card :body-style="{ padding: '20px' }">
            <el-row :gutter="20">
              <el-col
                :sm="24"
                :md="12"
              >
                <h3>Form</h3>
                <el-form
                  label-position="left"
                  label-width="100px"
                  :model="formModel"
                >
                  <el-form-item label="month">
                    <el-select
                      v-model="monthModel"
                      placeholder="Select"
                      @change="onChange"
                    >
                      <el-option
                        v-for="item in options"
                        :key="item.id"
                        :label="item.value"
                        :value="item.id"
                      >
                      </el-option>
                    </el-select>
                  </el-form-item>
                  <el-form-item label="day">
                    <el-select
                      v-model="dayModel"
                      placeholder="Select"
                    >
                      <el-option
                        v-for="item in days"
                        :key="item.value"
                        :label="item.value"
                        :value="item"
                      >
                      </el-option>
                    </el-select>
                  </el-form-item>
                  <el-form-item label="kilometers">
                    <el-input
                      type="number"
                      v-model="formModel.km"
                    ></el-input>
                  </el-form-item>
                  <el-form-item label="address">
                    <el-input v-model="formModel.addressTo"></el-input>
                  </el-form-item>
                </el-form>
              </el-col>
              <el-col
                :sm="24"
                :md="12"
              >
                <h3>Total</h3>
                <el-form
                  label-position="left"
                  label-width="100px"
                >
                  <el-form-item label="Kilometers">
                    <span>{{getAllKm}}</span>
                  </el-form-item>
                  <el-form-item label="Rate">
                    <el-input
                      type="number"
                      v-model="rate"
                    ></el-input>
                  </el-form-item>
                  <el-form-item label="Result">
                    <span>{{getResult}}</span>
                  </el-form-item>
                </el-form>
              </el-col>
            </el-row>
            <el-row
              type="flex"
              justify="end"
            >
              <div>
                <el-button @click="clearForm">
                  Clear
                </el-button>
                <el-button
                  type="success"
                  @click="addItem"
                >Add Day</el-button>
                <el-button
                  :disabled="!tableData.length"
                  type="primary"
                  @click="downoloadPdf"
                >Download PDF</el-button>
              </div>
            </el-row>
          </el-card>
        </div>
      </el-col>
    </el-row>
    <el-row
      type="flex"
      justify="center"
    >
      <el-col :span="16">
        <div>
          <el-table
            :data="tableData"
            style="width: 100%"
          >
            <el-table-column
              prop="week"
              label="Week"
              width="180"
            >
            </el-table-column>
            <el-table-column
              prop="date"
              label="Datum"
              width="180"
            >
            </el-table-column>
            <el-table-column
              prop="km"
              label="Kilometers"
              width="180"
            >
            </el-table-column>
            <el-table-column
              prop="addressFrom"
              label="Vertrekadres"
            >
            </el-table-column>
            <el-table-column
              prop="addressTo"
              label="Aankomstadres"
            >
            </el-table-column>
          </el-table>
        </div>
      </el-col>
    </el-row>
    <div
      id="pdfHtml"
      style="display: none"
    >
      <p>Bedrijfsnaam KW Super Schoon</p>
      <p>Adres Valkhof 159</p>
      <p>Postcode & plaats 3362GG Sliedrecht</p>

    </div>
    <div
      id="editor"
      style="display: none;"
    >
      <p>Merk auto: Skoda</p>
      <p>Type autoo: Fabia</p>
      <p>Kenteken auto: 05-SZ-LK</p>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import moment from 'moment';
import * as jsPDF from "jspdf";
import 'jspdf-autotable';

@Component({
  components: {
  }
})
export default class Home extends Vue {


  ADDRESS: string = '3362GG Sliedrecht';
  PDF_HEAD: Array<Array<string>> = [["Week", "Datum", "Kilometers", "Vertrekadres", "Aankomstadres"]];
  formModel: Object = { date: '', km: '', week: '', addressFrom: this.ADDRESS, addressTo: '' };
  days: Array<any> = [];
  monthModel: string = '';
  options: Array<Object> = moment.months().map((date, index) => ({ id: index, value: date }));
  data: Array<any> = [];
  tableData: Array<any> = [];
  dayModel: any = '';
  daysMap: any = {};
  rate: number = 0.19;
  result: number = 0;

  get getDays() {
    return this.days;
  }

  get getFormValidation() {
    return Object.values(this.formModel).every(val => !!val);
  }

  get getAllKm() {
    let sum = 0;
    this.tableData.map(el => {
      sum += parseInt(el.km);
    });
    return sum.toFixed(2);
  }

  get getResult() {
    return (this.getAllKm * this.rate).toFixed(2);
  }

  onChange(val: any) {
    this.daysMap = this.createDays(val);
  }

  createDays(month: number): any {
    let date = moment(new Date(2019, month, 1));
    let daysCount = date.daysInMonth();
    let days: any = {};
    for (let i = 1; i <= daysCount; i++) {
      let dateObj = moment(new Date(2019, month, i));
      let weekNumber = dateObj.week();
      let stringDate = dateObj.format('MMMM DD,YYYY');
      let stringDay = moment.weekdays(i);
      if (!days[weekNumber]) days[weekNumber] = [];
      days[weekNumber].push(`${stringDay}, ${stringDate}`);
      this.days.push({ value: `${stringDay}, ${stringDate}`, week: weekNumber });
    }
    return days;
  }

  clearForm() {
    this.formModel = { addressFrom: '3362GG Sliedrecht' };
    this.dayModel = {};
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
    this.days.map(day => {
      let arr = [];
      arr.push(day.week);
      const item = this.tableData.find(el => el.date === day.value);
      if (item) {
        arr.push(item.date);
        arr.push(item.km);
        arr.push(item.addressFrom);
        arr.push(item.addressTo);
      } else {
        arr.push(day.value);
        arr.push('');
        arr.push(this.ADDRESS);
        arr.push('');
      }
      this.data.push([...arr]);
    });
  }

  downoloadPdf() {
    this.createPdfContent();
    const doc = new jsPDF();

    let id = document.querySelector('#pdfHtml');
    let editor = document.querySelector('#editor');
    doc.fromHTML(id, 14, 10, {});
    doc.text('Periode: Mei 2019', 150, 10);
    doc.fromHTML(editor, 14, 40, { bottom: 50 });
    doc.autoTable({ showHead: 'firstPage', startY: 80, head: this.PDF_HEAD, body: this.data });
    doc.autoTable({ head: [['Kilometers', 'Rate', 'Result']], body: [[this.getAllKm, this.rate, this.getResult]] })
    doc.output("dataurlnewwindow");
    // doc.save('test.pdf');
  }
}
</script>
<style lang="scss" scoped>
.home {
  width: 100%;
  padding: 0 50px;
}

.el-select {
  width: 100%;
}
</style>
