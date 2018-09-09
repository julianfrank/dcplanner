<template>
<div>
    <h3>Rack Status</h3>
<table align="center">
<thead>
    <th>Server Type</th>
    <th v-for="(rack,rackIndex) in RackLabel" :key="rackIndex">
        {{rack}}
        </th>
        <th>Row Count</th>
        <th>Row Power</th>
        <th>Row Space</th>
</thead>
<tbody>
<tr v-for="(svr,svrIndex) in ServerLabel" :key="svrIndex">
    <td>{{svr}}</td>
    <td v-for="(n,nIndex) in N[svrIndex]" :key="nIndex">
        {{n}}
    </td>
    <td>{{RowCount(svrIndex)}}</td>
    <td>{{RowPower(svrIndex)}}</td>
    <td>{{RowSpace(svrIndex)}}</td>
</tr>
<tr>
    <td>Col Count</td>
    <td v-for="(col,i) in N[0]" :key="i">{{ColCount(i)}}</td>
</tr>
<tr>
    <td>Col Power</td>
    <td v-for="(col,i) in N[0]" :key="i">{{ColPower(i)}}</td>
</tr>
<tr>
    <td>Col Space</td>
    <td v-for="(col,i) in N[0]" :key="i">{{ColSpace(i)}}</td>
</tr>
</tbody>
</table>
<br>
<button @click="this.doIteration">Perform 1 Iteration</button>
<button>Stop / Pause</button>

</div>    
</template>

<script>
export default {
  data() {
    return {
      //Inputs
      ServerLabel: ["S1", "S2", "S3"],
      ServerPower: [2000, 3000, 5000],
      ServerSpace: [2, 3, 5],
      ServerCount: [7, 5, 3],
      RackLabel: ["R1", "R2", "R3", "R4", "R5"],
      RackPower: [10000, 9000, 7000, 5000, 3000],
      RackSpace: [10, 9, 7, 5, 3],
      //Outputs
      N: [[]] //rows=>Server type,columns=>Rack Position
    };
  },
  mounted() {
    //Initialize to all 0
    this.N = this.ServerLabel.map((_, row) =>
      this.RackLabel.map(() => this.ServerCount[row])
    );
  },
  methods: {
    RowCount(i) {
      let row = this.N[i] || [],
        sum = row.reduce((a, b) => a + b, 0),
        input = this.ServerCount[i],
        delta = sum - input;
      return { input, sum, delta };
    },
    RowPower(i) {
      let row = this.N[i] || [],
        sum = this.ServerPower[i] * this.RowCount(i).sum,
        input = this.ServerPower[i] * this.RowCount(i).input,
        delta = sum - input;
      return { input, sum, delta };
    },
    RowSpace(i) {
      let row = this.N[i] || [],
        sum = this.ServerSpace[i] * this.RowCount(i).sum,
        input = this.ServerSpace[i] * this.RowCount(i).input,
        delta = sum - input;
      return { input, sum, delta };
    },
    ColCount(i) {
      let col = this.N.map(row => row[i]) || [],
        sum = col.reduce((a, b) => a + b, 0),
        input = this.RackSpace[i],
        delta = sum - input;
      return { input, sum, delta };
    },
    ColPower(i) {
      let col = this.N.map(row => row[i]) || [],
        sum = col.reduce((a, b, row) => a + b * this.ServerPower[row], 0),
        input = this.RackPower[i],
        delta = sum - input;
      return { input, sum, delta };
    },
    ColSpace(i) {
      let col = this.N.map(row => row[i]) || [],
        sum = col.reduce((a, b, row) => a + b * this.ServerSpace[row], 0),
        input = this.RackSpace[i],
        delta = sum - input;
      return { input, sum, delta };
    },
    doIteration() {
      this.N = this.N.map((rowValue, row) => {
        return rowValue.map((val, col) => {
          let deltaCount = this.RowCount(row).delta,
            deltaPower = this.ColPower(col).delta,
            deltaSpace = this.ColSpace(col).delta;
          console.log(row, col, val, deltaCount, deltaPower, deltaSpace);
          if (deltaCount < 0) {
            return val + 1;
          } else if (deltaPower > 0 && deltaSpace > 0) {
            return val == 0 ? val : val - 1;
          } else if (deltaCount == 0) {
            return val;
          } else {
            return val;
          }
        });
      });
    }
  }
};
</script>
