<template>
  <div>
    <div class="step">

      <div class="step-text">
        <h2>STEP 1</h2>
      </div>
      <div class="step-info">
        <div class="table-holder">
          <span>Agents:</span>
          <table>
            <tr>
              <th>Name</th>
              <th>Budget</th>
              <th>Actives</th>
              <th>Interest</th>
              <th>Sold To</th>
            </tr>
            <tr v-for="agent in agents" :key="agent._id">
              <td>{{agent.name}}</td>
              <td>{{agent.budget}} $</td>
              <td>
                <ul>
                  <li v-for="activ in agent.activities" :key="activ.id">
                    {{activ.id  }}
                  </li>
                </ul>
              </td>
              <td>{{agent.interest}}%</td>
              <td>{{agent.sold_to}}</td>
            </tr>
          </table>
        </div>
        <div class="table-holder">
          <span>Memory and Stor Cost:</span>
          <table>
            <tr>
              <th>Name</th>
              <th>Memory</th>
              <th>Memory Cost</th>
              <th>Stor Cost</th>
            </tr>
            <tr v-for="cost in otherCost" :key="cost._id">
              <td>{{cost.name}}</td>
              <td>{{cost.memory}} GB</td>
              <td>{{cost.memoryCost}} $</td>
              <td>{{cost.storCost}} $</td>
            </tr>
          </table>
        </div>
        <div class="table-holder">
          <span>TransferCost:</span>
          <table>
            <tr>
              <th>Country</th>
              <th>Transfer Cost</th>
            </tr>
            <tr v-for="region in regions" :key="region._id">
              <td>{{region.name}}</td>
              <td>{{region.transferCost}} $</td>
            </tr>
          </table>
        </div>
        <div class="table-holder">
          <span>Machines:</span>
          <table>
            <tr>
              <th>Name</th>
              <th>Memory</th>
              <th>Memory Cost</th>
              <th>Stor Cost</th>
              <th>Region</th>
              <th>Transfer Cost</th>
              <th>Penalty Cost</th>
              <th>Total Cost</th>
            </tr>
            <tr v-for="vm in virtualMachines" :key="vm._id">
              <td>{{vm.name}}</td>
              <td>{{vm.memory}} GB</td>
              <td>{{vm.memoryCost}} $</td>
              <td>{{vm.storCost}} $</td>
              <td>{{vm.fromRegion}}</td>
              <td>{{vm.transferCost}} $</td>
              <td>{{vm.penaltyCost}} $</td>
              <td>{{vm.totalCost}} $</td>
            </tr>
          </table>
        </div>
      </div>

    </div>

    <div class="step">
      <div class="step-text">
        <h2>STEP 2</h2>
      </div>
      <button class="buy_machines" v-on:click="buyMachines">BUY MACHINES</button>

      <div class="step-info ">
      </div>
      
    </div>

  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data() {
    return {
      agents: [],

      virtualMachines: [],

      regions: [],

      otherCost: [],

      memoryCostForOneGB: 15,
      storCostForOneGB: 10,
      penaltyCost: 0.05,

      step2: false
    };
  },
  mounted() {
    this.fillAgents();
    this.fillRandomInfo();
  },
  methods: {
    getRandomInt(min, max) {
      return Math.floor(Math.random() * (max - min) + min);
    },
    fillRandomInfo() {
      for(let i = 0; i < 3; i++) {
        this.regions.push({
          id: i,
          name: `REGION${i+1}`,
          transferCost: this.getRandomInt(10, 200),
        });
      }
      for(let i = 0; i < 5; i++) {
        this.otherCost.push({
          id: i,
          name: `DC${i+1}`,
          memory: this.getRandomInt(2, 32),
          memoryCost: 0,
          storCost: 0
        });
      }
      (this.otherCost).sort((a, b) => (a.memory < b.memory) ? 1 : -1);

      for(let i = 0; i < (this.otherCost).length; i++) {
        this.otherCost[i].memoryCost = this.otherCost[i].memory * this.memoryCostForOneGB;
        this.otherCost[i].storCost = this.otherCost[i].memory * this.storCostForOneGB; 
      }

      for(let i = 0; i < (this.otherCost).length; i++) {
        let temp_reg = this.getRandomInt(0, 3);
        this.virtualMachines.push({
          id: i,
          name: `VM${i+1}`,
          memory: this.otherCost[i].memory,
          memoryCost: this.otherCost[i].memoryCost,
          storCost: this.otherCost[i].storCost,
          fromRegion: this.regions[temp_reg].name,
          transferCost: this.regions[temp_reg].transferCost,
          penaltyCost: 0,
          totalCost: 0,
          sold: false
        })
      }

      for(let i = 0; i < (this.virtualMachines).length; i++) {
        let total_temp = this.virtualMachines[i].memoryCost + this.virtualMachines[i].storCost + this.virtualMachines[i].transferCost;
        this.virtualMachines[i].penaltyCost = Math.ceil(total_temp * this.penaltyCost);
        this.virtualMachines[i].totalCost = this.virtualMachines[i].penaltyCost + total_temp;
      }

    },
    fillAgents() {
      for(let i = 0; i < 10; i++) {
        this.agents.push({
          id: i,
          name: `AGENT${i+1}`,
          budget: this.getRandomInt(1000, 2000),
          actives: [],
          interest: this.getRandomInt(10, 20),
          sold_to: []
        });
      }
      (this.agents).sort((a, b) => (a.budget > b.budget) ? 1 : -1);
    },

    buyMachines() {
      for(let i = 0; i < (this.agents).length; i++) {
        let remain_budget = this.agents[i].budget;
        for(let j = 0; j < (this.virtualMachines).length; j++) {
          if(this.virtualMachines[j].totalCost < remain_budget && !this.virtualMachines[j].sold) {
            this.agents[i].actives.push({
              id: j,
              vmName: this.virtualMachines[j].name,
              vmInitCost: this.virtualMachines[j].totalCost,
              vmBoughtCost: this.virtualMachines[j].totalCost,
              vmCost: this.virtualMachines[j].totalCost * ((this.agents[i].interest / 100) + 1)
            })
            this.virtualMachines[j].sold = true;
            remain_budget = remain_budget - this.virtualMachines[j].totalCost;
            continue;
          }
        } 
      }
    }

   
  }
}
</script>

<style>
table {
  border: 1px solid grey;
}
table td, th {
  padding: 10px;
}
.step {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.step-info {
  display: flex;
}
.table-holder {
  margin: 20px;
}
.buy_machines {
  border: none;
  background-color: rgb(50, 36, 73);
  color: white;
  padding: 20px;
  cursor: pointer;
  width: 200px;
}
</style>
