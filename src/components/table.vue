<template>
  <k-field :disabled="disabled" :label="label" :name="name" :required="required" :type="type" class="k-table-field" v-bind="$props">
    <div class="table-field field field-name" v-cloak data-field='table'>
      <!-- Column Controls -->
      <div class="table-ctrl">
        <div class="row-cell column-ctrl" v-for="n in columnCount">
          <div class="centering">
            <k-button icon="angle-left" v-bind:class="{ disabled: n == 1 }" @click="moveColumn(n-1, 'left')"></k-button>
            <k-button icon="remove" v-bind:class="{ disabled: columnCount <= minColumns }" @click="deleteColumn(n)"></k-button>
            <k-button @click="alignColumn(n-1, 'left')">L</k-button>
            <k-button @click="alignColumn(n-1, 'center')">C</k-button>
            <k-button @click="alignColumn(n-1, 'right')">P</k-button>
            <k-button icon="bold" @click="makeColumnBold(n-1)"></k-button>
            <k-button icon="angle-right" v-bind:class="{ disabled: n == columnCount }"  @click="moveColumn(n-1, 'right')"></k-button>
          </div>
        </div>
        <div class="row-ctrl add-column">
          <k-button icon="add" v-bind:class="{ disabled: columnCount >= maxColumns }" @click="addColumn()"></k-button>
        </div>
      </div>
      <!-- Table Body -->
      <div class="table-container">
        <!-- Row -->
        <div class="table-row" v-for="(row, rowIndex) in betterVal">
          <div class="row-ctrl move-row">
            <k-button icon="angle-up" v-bind:class="{ disabled: rowIndex == 0 }" @click="moveRow(rowIndex, 'up')"></k-button>
            <k-button icon="angle-down" v-bind:class="{ disabled: rowIndex == rowCount-1 }" @click="moveRow(rowIndex,	'down')"></k-button>
          </div>
          <input class="row-cell input"
                 v-bind:class="{
                   bold: row[cellIndex].bold,
                   'text-left': row[cellIndex].alignment === 'left',
                   'text-center': row[cellIndex].alignment === 'center',
                   'text-right': row[cellIndex].alignment === 'right',
                 }"
                 :name="'name[table]['+ rowIndex +']'"
                 v-model="row[cellIndex].value"
                 v-on:change="updateTable()"
                 v-for="(cell, cellIndex) in row"
          />
          <div class="row-ctrl delete-row">
            <k-button icon="remove" @click="deleteRow(rowIndex)" v-show="rowCount > 1"></k-button>
          </div>
        </div>
      </div>
      <!-- Add Row Btn-->
      <div class="table-add-row">
        <div class="row-cell">
          <k-button icon="add" @click="addRow()">Add</k-button>
        </div>
      </div>
    </div>
  </k-field>
</template>

<script>
  import _ from '../assets/js/lodash.js'
  export default {
    props: {
      minColumns: Number,
      maxColumns: Number,

      // general options
      label: String,
      disabled: Boolean,
      help: String,
      parent: String,
      value: [String, Array],
      name: [String, Number],
      required: Boolean,
      type: String
    },
    created: function () {
      this.value = this.getData(this.betterVal, [_.fill(Array(this.minColumns), '')]);
    },
    computed: {
      columnCount: function () {
        if(this.betterVal == null){
          return this.minColumns;
        } else {
          return this.betterVal[0].length;
        }
      },
      rowCount: function () {
        return this.betterVal.length;
      },
      // Better control of this.value, always use this.betterval
      betterVal: function () {
        let newValue;
        if (typeof this.value === 'string') {
          let spittedString = this.value.split('\n');
          const selfmadeValue = [];
          let row = [];
          spittedString.forEach((val) => {
            if (val == '- ') {
              if (row.length > 0) {
                selfmadeValue.push(row);
              }
              row = [];
            } else {
              row.push(val.split('- ').pop().replace(/\""/g, ""));
            }
          });
          selfmadeValue.push(row);
          newValue = selfmadeValue;
        } else {
          newValue = this.value;
        }
        if(newValue == null){
          newValue = new Array();
          let insideValue = new Array();
          for(let i = 0; i < this.minColumns; i++){
            const emptyCell = this.emptyCell();
            insideValue.push(emptyCell);
          }
          newValue.push(insideValue);
        }
        return newValue;
      }
    },
    methods: {
      emptyCell: function() {
        return {
          value: '',
          bold: false,
        }
      },
      getData: function (value, defaultValues) {
        if (_.isEmpty(value)) {
          value = defaultValues;
        }
        return value;
      },
      addRow: function () {
        const newRow = [];
        for (let i = 0; i < this.columnCount; i++) {
          const cell = this.emptyCell();
          if (this.rowCount === 0) {
            newRow[i] = cell;
            continue;
          }
          const previousCol = this.betterVal[this.rowCount - 1][i];
          cell.bold = previousCol.bold;
          cell.alignment = previousCol.alignment;
          newRow[i] = cell;
        }
        this.betterVal.push(newRow);
        this.updateTable();
      },
      deleteRow: function (rowNum) {
        this.betterVal.splice(rowNum, 1);
        this.updateTable();
      },
      moveRow: function (rowIndex, direction) {
        if (direction == 'up') {
          direction = -1;
        } else if (direction == 'down') {
          direction = 1;
        } else {
          console.error('Wrong direction!')
        }

        if (!((rowIndex == 0 && direction == -1) || ((rowIndex == (this.rowCount - 1)) && direction == 1))) {
          var changing = this.betterVal[rowIndex];
          this.betterVal.splice(rowIndex, 1);
          this.betterVal.splice(rowIndex + direction, 0, changing);
        }
        this.updateTable();
      },
      addColumn: function () {
        const emptyCell = this.emptyCell();
        _.forEach(this.betterVal, function (value) {
          value.push(emptyCell);
        });
        this.updateTable();
      },
      deleteColumn: function (colNum) {
        _.forEach(this.betterVal, function (value) {
          value.splice(colNum - 1, 1);
        });
        this.updateTable();
      },
      moveColumn: function (colNum, direction) {
        if (direction == 'left') {
          direction = -1;
        } else if (direction == 'right') {
          direction = 1;
        } else {
          console.error('Wrong direction!');
        }
        if (!((colNum == 0 && direction == -1) || ((colNum + 1 == this.columnCount) && direction == 1))) {
          _.forEach(this.betterVal, function (value) {
            value.splice(colNum + direction, 0, value.splice(colNum, 1)[0]);
          });
        }
        this.updateTable();
      },
      makeColumnBold: function (colNum) {
        if (this.rowCount === 0 || this.columnCount < colNum) {
          return;
        }
        let columnIsBold = true;
        _.forEach(this.betterVal, function (value) {
          let cellValue = value[colNum];
          if (cellValue && !cellValue.bold) {
            columnIsBold = false;
          }
        });
        _.forEach(this.betterVal, function (value) {
          let cellValue = value[colNum];
          if (cellValue) {
            value[colNum].bold = !columnIsBold;
          }
        });
        this.updateTable();
      },
      alignColumn: function(colNum, alignment) {
        if (this.rowCount === 0 || this.columnCount < colNum) {
          return;
        }
        _.forEach(this.betterVal, function (value) {
          let cellValue = value[colNum];
          if (cellValue) {
            value[colNum].alignment = alignment;
          }
        });
        this.updateTable();
      },
      updateTable: function () {
        this.$emit('input', this.betterVal);
      }
    }
  };
</script>

<style lang="scss">
  @import '../assets/scss/index.scss';
</style>
