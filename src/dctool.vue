<template>
  <div>
    
    <div hidden>
    racks:{{racks}}<br>
    servers:{{servers}}<br>
    serverCaps:{{serverCaps}}<br>
    serverCounts:{{serverCounts}}<br>
    solution:{{solution}}<br>
    input:{{input}}<br>
    output:{{output}}<br>
    </div>

<h1> DC Top Up Tool</h1><h4>Version:{{appVersion}}</h4>
<p>Use this tool to quickly plan which servers to place in which Pre-Occupied Rack</p>

Import:<input @change="importConfig" type="file">

<table>
  <caption>Rack Inputs</caption>
  <tr>
    <th>RUs Available</th>
    <th  v-for="(rack,ind) in racks" :key="ind" >Rack{{rack}}</th>
    <th>Add Rack<br>Change to add this Rack</th>
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
    <th>Add Server Type<br>Change Value to add new Server Type</th>
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
<button :disabled="busy" @click="evaluate">Evaluate</button>
<br>
<a  id="jsonExport" ></a>
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
<br>
<p>This is an Open Source Project with code hosted in <a href="https://github.com/julianfrank/dcplanner">https://github.com/julianfrank/dcplanner</p>
<p>If you liked this please do Star this Project in Github</p>
<p>Also for any new features or issues you discover please raise an issue in Github</p>
<p>Thanks</p>
<p>Julian Frank</p>
<br>
<del>Known Issue -> In some Combination the Output show floats (3.5, 5.5 etc...)</del>Resolved as on 13thNov2018
  </div>
</template>

<script>
import * as solver from "javascript-lp-solver";

window.onerror = err => alert("Error=> " + JSON.stringify(err));

export default {
  data() {
    return {
      rackCaps: [10, 7, 5],
      newRackCaps: 0,
      serverCaps: [1, 2, 4],
      newServerCap: 0,
      serverCounts: [5, 3, 1],
      newServerCount: 0,
      solution: {},
      input: {},
      output: {},
      busy: false,
      appVersion: 181113
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
    exportConfig() {
      let exportPack = {
        rackCaps: this.rackCaps,
        serverCaps: this.serverCaps,
        serverCounts: this.serverCounts,
        output: JSON.parse(JSON.stringify(this.output)),
        appVersion: this.appVersion,
        exportDate: new Date()
      };

      var jsonse = JSON.stringify(exportPack);
      var blob = new Blob([jsonse], { type: "application/json" });
      var url = URL.createObjectURL(blob);

      var a = document.getElementById("jsonExport");
      a.href = url;
      a.download = `JF dcplanner ${new Date()}.json`;
      a.textContent = `Download -> JF dcplanner ${new Date()}.json`;
    },

    importConfig(event) {
      let filesSelected = event.target.files;

      if (filesSelected.length > 0) {
        let file = filesSelected[0];
        if (file.type == "application/json") {
          let fr = new FileReader();
          fr.onload = e => {
            let data = JSON.parse(e.target.result);
            if (data.appVersion != this.appVersion)
              alert("Looks like this was created in a previous version...");
            this.rackCaps = data.rackCaps;
            this.serverCaps = data.serverCaps;
            this.serverCounts = data.serverCounts;
            this.output = data.output;
            alert("Imported");
          };
          fr.readAsText(file);
        } else {
          alert(`Wrong File Type(${file.type}). Please select a JSON file`);
        }
      }
    },

    addRack() {
      this.rackCaps.push(this.newRackCaps);
      this.newRackCaps = 0;
      this.busy = false;
    },
    addServerCap() {
      this.serverCaps.push(this.newServerCap);
      this.newServerCap = 0;
      if (this.serverCaps.length > this.serverCounts.length)
        this.serverCounts.push(this.newServerCount);
      this.busy = false;
    },
    addServerCount() {
      this.serverCounts.push(this.newServerCount);
      this.newServerCount = 0;
      if (this.serverCounts.length > this.serverCaps.length)
        this.serverCaps.push(this.newServerCap);
      this.busy = false;
    },

    evaluate() {
      if (this.racks < 1 || this.servers < 1) return;

      let model = [];

      this.rackCaps.forEach((rackCap, rackInd) => {
        let rackConstr = "";
        this.serverCaps.forEach((modelCap, modelInd) => {
          //Add Integer Constraints
          model.push(`int ps${modelInd}.n${rackInd}.${modelInd}`);
          //Add Model level Constraint
          rackConstr += `${modelCap} ps${modelInd}.n${rackInd}.${modelInd} `;
        });
        model.push(`${rackConstr} <= ${rackCap}`);
      });
      let maxConstr = "max: ";
      this.serverCounts.forEach((modelCount, modelInd) => {
        let modelConstr = "";
        this.rackCaps.forEach((rackCap, rackInd) => {
          modelConstr += `${
            this.serverCaps[modelInd]
          } ps${modelInd}.n${rackInd}.${modelInd} `;
          maxConstr += `${modelCount}  ps${modelInd}.n${rackInd}.${modelInd} `;
        });
        model.push(
          `${modelConstr} = ${modelCount * this.serverCaps[modelInd]}`
        );
      });
      model.push(maxConstr);

      // Reformat to JSON model
      model = solver.ReformatLP(model);

      let results = solver.Solve(model);

      if (results.feasible) {
        this.output = results;
        this.exportConfig();
      } else {
        alert(" Solution NOT Feasible!");
      }
    },
    outputMat() {
      let mat = [];
      for (let s = 0; s < this.servers; s++) {
        let row = [];
        for (let r = 0; r < this.racks; r++) {
          let label = `ps${s}.n${r}.${s}`;
          row.push(this.output[label]);
        }
        mat.push(row);
      }
      return mat;
    }
  }
};
</script>


