<template>
  <div class="wapper">
    <div class="textfleld">
      <img @click="handle_search" :src="icon" alt="" />
      <input focus type="number" @keyup.enter="handle_search" v-model="port" />
    </div>
    <div class="contant">
      <div v-if="!loading">
        <div v-for="(tds, i) in merge_lists.tds" :key="i">
          <div class="row" v-if="tds.TD_PID">
            <div>
              <span class="pid">{{ tds.TD_PID }}</span>
              <button class="close" @click="handle_kill(tds.TD_PID)">
                kill
              </button>
            </div>
            <div>
              <span class="user">{{ tds.TD_USER }}</span>
              <span class="command"> {{ tds.TD_COMMAND }}</span>
              <span class="command"> {{ tds.TD_DEVICE }}</span>
            </div>
            <div class="name">name:{{ tds.TD_NAME }}</div>
          </div>
        </div>
      </div>
      <div v-else>
        <div class="spinner">
          <div class="double-bounce1"></div>
          <div class="double-bounce2"></div>
        </div>
      </div>

      <!-- <table>
        <thead class="thead">
          <tr>
            <th class="th" v-for="(trs, j) in merge_lists.trs" :key="j">
              {{ trs }}
            </th>
          </tr>
        </thead>
        <tbody class="tbody">
          <tr v-for="(tds, i) in merge_lists.tds" :key="i">
            <td class="td" v-for="(td, k) in tds" :key="k">
              {{ td }}
            </td>
          </tr>
        </tbody>
      </table> -->
    </div>
  </div>
</template>

<script>
const exec = window.require("child_process").exec;
const search_all = "lsof -i";
export default {
  name: "App",
  data() {
    return {
      port: 8080,
      list: [],
      icon: require("./assets/search.svg"),
      loading: false,
    };
  },
  computed: {
    merge_lists() {
      let result = {};
      if (this.list.length) {
        const [tr, ...tds] = this.list.split("\n");
        const [
          COMMAND,
          PID,
          USER,
          FD,
          TYPE,
          DEVICE,
          SIZE_OFF,
          NODE,
          NAME,
        ] = tr.split(/\s+/);
        result.trs = [
          PID,
          COMMAND,
          USER,
          NODE,
          TYPE,
          DEVICE,
          SIZE_OFF,
          FD,
          NAME,
        ];
        result.tds = tds.map((e) => {
          const [
            TD_COMMAND,
            TD_PID,
            TD_USER,
            TD_FD,
            TD_TYPE,
            TD_DEVICE,
            TD_SIZE_OFF,
            TD_NODE,
            TD_NAME,
          ] = e.split(/\s+/);
          return {
            TD_PID,
            TD_COMMAND,
            TD_USER,
            TD_NODE,
            TD_TYPE,
            TD_DEVICE,
            TD_SIZE_OFF,
            TD_FD,
            TD_NAME,
          };
        });
      }
      return result;
    },
  },
  mounted() {
    this.handle_all_port(search_all);
  },
  components: {},
  methods: {
    async_search(command) {
      this.loading = true;
      exec(command, (err, strout) => {
        this.list = strout;
        this.loading = false;
      });
    },
    handle_kill(port) {
      exec(`kill -9 ${port}`, (err, strout) => {
        console.log(err);
        if (strout) {
          this.handle_all_port();
        }
      });
    },
    handle_all_port() {
      this.async_search(search_all);
    },
    handle_search() {
      const command = this.port ? `lsof -i tcp:${this.port}` : search_all;
      this.async_search(command);
    },
  },
};
</script>

<style >
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  -webkit-app-region: drag;
}

.wapper {
  height: 580px;
  width: 290px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}
.textfleld {
  width: 98%;
  display: flex;
  justify-content: center;
  align-items: center;
  border: 1px solid #efefef;
  border-radius: 5px;
  overflow: hidden;
  background: #fff;
}
.textfleld input {
  height: 30px;
  flex: 1;
  outline: none;
  border: none;
  padding-right: 10px;
}
.textfleld img {
  height: 20px;
  width: 20px;
  margin: 0 10px;
  cursor: pointer;
}
.contant {
  margin-top: 10px;
  overflow: scroll;
  flex: 1;
  width: 100%;
}
.row {
  background-color: #fff;
  margin: 0 10px 10px;
  border-radius: 5px;
  padding: 10px 15px;
  font-size: 12px;
}
.row div {
  margin-top: 8px;
}
.pid {
  background-color: #000;
  color: #fff;
  font-size: 12px;
  padding: 3px 5px;
  border-radius: 3px;
  font-weight: bold;
}
.close {
  margin-left: 10px;
}
.command {
  margin-left: 10px;
}
.name {
  font-size: 12px;
  word-break: break-all;
}
.spinner {
  width: 60px;
  height: 60px;

  position: relative;
  margin: 100px auto;
}

.double-bounce1,
.double-bounce2 {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background-color: #999;
  opacity: 0.6;
  position: absolute;
  top: 0;
  left: 0;

  -webkit-animation: bounce 2s infinite ease-in-out;
  animation: bounce 2s infinite ease-in-out;
}

.double-bounce2 {
  -webkit-animation-delay: -1s;
  animation-delay: -1s;
}

@-webkit-keyframes bounce {
  0%,
  100% {
    -webkit-transform: scale(0);
  }
  50% {
    -webkit-transform: scale(1);
  }
}

@keyframes bounce {
  0%,
  100% {
    transform: scale(0);
    -webkit-transform: scale(0);
  }
  50% {
    transform: scale(1);
    -webkit-transform: scale(1);
  }
}
</style>
