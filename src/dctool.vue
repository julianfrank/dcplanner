<template>
  <div>
    
<table>
  <caption>Rack Inputs</caption>
  <tr>
    <th>RUs Available</th>
    <th  v-for="(rack,ind) in racks" :key="ind" >Rack{{rack}}</th>
    <th>Rack{{racks+1}} <br>Change to add this Rack</th>
  </tr>
  <tr>
    <td>RU in each Rack</td>
    <td v-for="(rack,ind) in racks" :key="ind"><input type="number" min="0" max="24" v-model.number="rackCaps[ind]"/></td>
    <td><input type="number" min="0" max="24" v-model.number="newRackCaps" @change="addRack"/></td>
  </tr>
</table>
<br>

<table>
  <caption>Server Inputs</caption>
  <tr>
    <th>Specs</th>
    <th  v-for="(server,ind) in servers" :key="ind">Server Type {{ind+1}}</th>
    <th>Server Type {{servers+1}}<br>Change Value to add new Server Type</th>
  </tr>
  <tr>
    <td>Count</td>
    <td v-for="(server,ind) in serverCounts" :key="ind"><input type="number" min="0" max="99" v-model.number="serverCounts[ind]"/></td>
    <td><input type="number" min="0" max="99" v-model.number="newServerCount" @change="addServerCount"/></td>
  </tr>
  <tr>
    <td>RUs per Server</td>
    <td v-for="(server,ind) in serverCaps" :key="ind"><input type="number" min="1" max="12" v-model.number="serverCaps[ind]"/></td>
    <td><input type="number" min="1" max="12" v-model.number="newServerCap" @change="addServerCap"/></td>
    </tr>
</table>
<br>
<button @click="evaluate">Evaluate</button>
<br>
<table>
  <caption>Output</caption>
  <tr>
    <th>Output</th>
    <th  v-for="(rack,ind) in racks" :key="ind">Rack{{rack}}</th>
  </tr>
  <tr v-for="(row,r_ind) in outputMat()" :key="r_ind">
    <td>Server Type {{r_ind+1}}</td>
    <td v-for="(val,c_ind) in row" :key="c_ind">{{val}}</td>
  </tr>
</table>

  </div>
</template>

<script>
import Constrained from 'constrained';
var mySystem = new Constrained.System();

export default {
  data() {
    return {
      rackCaps: [13, 11, 7],
      newRackCaps: 0,
      serverCaps: [1, 2],
      newServerCap: 0,
      serverCounts: [7, 11],
      newServerCount: 0,
      solution: {},
      input: {},
      output: {}
    };
  },
  mounted() {},
  computed: {
    racks() {
      return Math.max(this.rackCaps.length, 0);
    },
    servers() {
      return Math.max(this.serverCaps.length, this.serverCounts.length, 0);
    }
  },
  methods: {
    addRack() {
      this.rackCaps.push(this.newRackCaps);
      this.newRackCaps = 0;
    },
    addServerCap() {
      this.serverCaps.push(this.newServerCap);
      this.newServerCap = 0;
      if (this.serverCaps.length > this.serverCounts.length) this.serverCounts.push(this.newServerCount);
    },
    addServerCount() {
      this.serverCounts.push(this.newServerCount);
      this.newServerCount = 0;
      if (this.serverCounts.length > this.serverCaps.length) this.serverCaps.push(this.newServerCap);
    },
    evaluate() {
      this.input = {};
      //Init Input object
      this.rackCaps.forEach((val, ind) => {
        let label = `r${ind + 1}cap`;
        this.input[label] = val;
        mySystem.addConstant(label, this.input, label);
      });
      this.serverCaps.forEach((val, ind) => {
        let label = `s${ind + 1}cap`;
        this.input[label] = val;
        mySystem.addConstant(label, this.input, label);
      });
      this.serverCounts.forEach((val, ind) => {
        let label = `s${ind + 1}count`;
        this.input[label] = val;
        mySystem.addConstant(label, this.input, label);
      });

      try {
        for (let s = 1; s <= this.servers; s++) {
          for (let r = 1; r <= this.racks; r++) {
            let label = `s${s}r${r}`;
            this.output[label] = 0;
            mySystem.addVariable(label, this.output, label);
            //Ensure solution count is 0 or +ve
            let cons = `${label} >=0`;
            console.debug('Going to add Constraint ', cons);
            mySystem.addConstraint(cons);
            //ensure count is integet :wink:[Not working ]
            //cons = `10000000000*(${label}+1)/10000000000/(${label}+1) = 1`;
            //console.debug("Going to add Constraint ", cons);
            //mySystem.addConstraint(cons);
          }
        }
        //Add constraints
        for (let s = 1; s <= this.servers; s++) {
          let cons = '';
          for (let r = 1; r <= this.racks; r++) {
            cons += `s${s}r${r}` + (r == this.racks ? ` = s${s}count"` : ' + ');
          }
          console.debug('Going to add Constraint ', cons);
          mySystem.addConstraint(cons);
        }
        for (let r = 1; r <= this.racks; r++) {
          let cons = '';
          for (let s = 1; s <= this.servers; s++) {
            cons += `s${s}r${r}*s${s}cap` + (s == this.servers ? ` <= r${r}cap` : ' + ');
          }
          console.debug('Going to add Constraint ', cons);
          mySystem.addConstraint(cons);
        }
        //Setup Solution Hooks
        mySystem.onNewSolution(solution => (this.solution = solution), this.output);
        //resolve
        mySystem.resolve();
      } catch (error) {
        alert('Solution NOT Feasible');
      }
    },
    outputMat() {
      let mat = [];
      for (let s = 1; s <= this.servers; s++) {
        let row = [];
        for (let r = 1; r <= this.racks; r++) {
          let label = `s${s}r${r}`;
          row.push(this.output[label]);
        }
        mat.push(row);
      }
      return mat;
    }
  }
};
</script>

<style>
</style>
