<template>
  <div class="home">
    <div class="container">
    <div class="panel panel-sm">
      <div class="panel-heading"> 
        <h4>CSV Import</h4>
      </div>
      <div class="panel-body">
        <div class="form-group">
          <label for="csv_file" class="control-label col-sm-3 text-right">CSV file to import</label>
          <div class="col-sm-9">
            <input type="file" id="csv_file" name="csv_file" class="form-control" @change="loadCSV($event)">
          </div>
        </div>
        <div class="col-sm-offset-3 col-sm-9">
          <div class="checkbox-inline">
            <label for="header_rows"><input type="checkbox" id="header_rows"> File contains header row?</label>
          </div>
        </div>
        
        <div class="col-sm-offset-3 col-sm-9">
          <a href="#" class="btn btn-primary">Parse CSV</a>
        </div>
        <table v-if="parse_csv">
          <thead>
            <tr>
              <th v-for="item in parse_header"
                  @click="sortBy(item)"
                  :class="{ active: sortKey == item }" :key="item.id">
                {{ item | capitalize }}
                <span class="arrow" :class="sortOrders[item] > 0 ? 'asc' : 'dsc'">
                </span>
              </th>
            </tr>
          </thead> 
          <tr v-for="csv in parse_csv" :key="csv.id">
            <td v-for="item in parse_header" :key="item.id">
              {{csv[item]}}
            </td>
          </tr>
          
        </table>
      </div>
    </div>
    
  </div>
  </div>
</template>

<script>
// @ is an alias to /src

export default {
  name: 'home',
  data: () => {
    return {
      channel_name: '',
      channel_fields: [],
      channel_entries: [],
      parse_header: [],
      parse_csv: [],
      sortOrders:{},
      sortKey: ''
    }
  },
  filters: {
    capitalize: function (str) {
      return str.charAt(0).toUpperCase() + str.slice(1)
    }
  },
  methods: {
    sortBy: function (key) {
      var vm = this
      vm.sortKey = key
      vm.sortOrders[key] = vm.sortOrders[key] * -1
    },
    csvJSON(csv){
      var vm = this
      var lines = csv.split("\n")
      var result = []
      var headers = lines[0].split(",")
      vm.parse_header = lines[0].split(",") 
      lines[0].split(",").forEach(function (key) {
        vm.sortOrders[key] = 1
      })
      
      lines.map(function(line, indexLine){
        if (indexLine < 1) return // Jump header line
        
        var obj = {}
        var currentline = line.split(",")
        
        headers.map(function(header, indexHeader){
          obj[header] = currentline[indexHeader]
        })
        
        result.push(obj)
      })
      
      result.pop() // remove the last item because undefined values
      
      return result // JavaScript object
    },
    loadCSV(e) {
      var vm = this
      if (window.FileReader) {
        var reader = new FileReader();
        reader.readAsText(e.target.files[0]);
        // Handle errors load
        reader.onload = function(event) {
          var csv = event.target.result;
          vm.parse_csv = vm.csvJSON(csv)
          
        };
        reader.onerror = function(evt) {
          if(evt.target.error.name == "NotReadableError") {
            alert("Canno't read file !");
          }
        };
      } else {
        alert('FileReader are not supported in this browser.');
      }
    }
  }
   
}
</script>
