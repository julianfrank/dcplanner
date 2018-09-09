<template>
<div>
    <h3>Rack Layout</h3>
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
    this.N = [[1, 2, 3, 4, 5], [11, 12, 13, 14, 15], [21, 22, 23, 24, 25]];
  },
  methods: {
    RowCount(i) {
      let row = this.N[i] || [],
        rowSum = row.reduce((a, b) => a + b, 0),
        inputCount = this.ServerCount[i],
        delta = rowSum - inputCount;
      return { inputCount, rowSum, delta };
    },
    RowPower(i) {
      let row = this.N[i] || [],
        rowPowerSum = this.ServerPower[i] * this.RowCount(i).rowSum,
        inputPower = this.ServerPower[i] * this.RowCount(i).inputCount,
        delta = rowPowerSum - inputPower;
      return { inputPower, rowPowerSum, delta };
    },
    RowSpace(i) {
      let row = this.N[i] || [],
        rowSpaceSum = this.ServerSpace[i] * this.RowCount(i).rowSum,
        inputSpace = this.ServerSpace[i] * this.RowCount(i).inputCount,
        delta = rowSpaceSum - inputSpace;
      return { inputSpace, rowSpaceSum, delta };
    },
    ColCount(i) {
      let col = this.N.map(row => row[i]) || [],
        colSum = col.reduce((a, b) => a + b, 0),
        inputCount = this.RackSpace[i],
        delta = colSum - inputCount;
      return { inputCount, colSum, delta };
    },
    ColPower(i) {
      let col = this.N.map(row => row[i]) || [],
        colPowerSum = col.reduce(
          (a, b, row) => a + b * this.ServerPower[row],
          0
        ),
        inputPower = this.RackPower[i],
        delta = colPowerSum - inputPower;
      return { inputPower, colPowerSum, delta };
    },
    ColSpace(i) {
      let col = this.N.map(row => row[i]) || [],
        colSpaceSum = col.reduce(
          (a, b, row) => a + b * this.ServerSpace[row],
          0
        ),
        inputSpace = this.RackSpace[i],
        delta = colSpaceSum - inputSpace;
      return { inputSpace, colSpaceSum, delta };
    }
  }
};
</script>
