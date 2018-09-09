<template>
<div>
    <h3>Rack Layout</h3>
<table>
<thead>
    <th>Server Type</th>
    <th v-for="(rack,rackIndex) in RackLabel" :key="rackIndex">
        {{rack}}
        </th>
</thead>
<tr v-for="(svr,svrIndex) in ServerLabel" :key="svrIndex">
    <td>{{svr}}</td>
    <td v-for="(n,nIndex) in N[svrIndex]" :key="nIndex">
        {{n}}
    </td>
</tr>
</table>
</div>    
</template>

<script>
export default {
  data() {
    return {
      //Inputs
      ServerLabel: ["S1", "S2", "S3"],
      ServerPower: [1000, 2000, 4000],
      ServerSpace: [1, 2, 4],
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
  }
};
</script>

<style>
</style>
