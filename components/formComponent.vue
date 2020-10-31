<template>
  <div class="form-container bg-gray-800 p-5">
    <div class="select-container">
      <span class="material-icons icon text-white">chevron_left</span>
      <select name="type"
        class="select bg-gray-800 text-white border-b border-white p-1"
        :class="{ 'gray': type === '' }"
        v-model="type">
        <option value="" disabled selected hidden>Type</option>
        <option v-for="opt in mokup" :value="opt" :key="opt">{{opt}}</option>
      </select>
    </div>
    <div class="select-container">
      <span class="material-icons icon tt text-white" @click="showTooltip('click')" @mouseover="showTooltip('over')">help_outline</span>
      <div class="tooltip text-white" v-if="tooltip">Start typing...
        <span
          class="material-icons icon clear text-white hover:text-gray-400"
          @click="tooltip = false">
          cancel
        </span>
      </div>
      <span
        class="material-icons icon add-form text-white hover:text-gray-400" @click.stop="addForm" v-show="showAdd">
        add_circle_outline
      </span>
      <input type="text" class="prediction bg-gray-800 text-white"
        v-model="address"
        placeholder="Address"
        :list="`addresses${listCounter}`"
        @keyup="call"
        :name="`addresses${listCounter}`">
      <datalist
        class="select prediction p-1"
        :class="{ 'gray': address === '' }"
        :id="`addresses${listCounter}`">
        <option v-for="opt in places" :value="opt.description" :key="opt.place_id">{{opt.description}}</option>
      </datalist>
      <label class="py-1 px-2 text-white" v-if="address !== '' && !errorMessage">Address</label>
      <label class="py-1 px-2 text-red-600" v-else>{{errorMessage}}</label>
    </div>
    <div class="select-container" @mouseover="showClear">
      <span
        class="material-icons icon clear text-white hover:text-gray-400"
        @click="clearNumber"
        @mouseover="showClear"
        v-if="mouseOverField && number !== ''">
        cancel
      </span>
      <input type="text"
        placeholder="Number"
        class="bg-gray-800 text-white"
        v-model="number"
        :disabled="address === ''"
        @mouseleave.once="appendNumber"
        :key="counter">
      <label class="py-1 px-2 text-white" v-if="number !== ''">No.</label>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      type: '',
      address: '',
      number: '',
      mouseOverField: false,
      mokup: [
        'Residential',
        'Domicile',
        'Legal',
        'Other'
      ],
      places: [],
      showAdd: false,
      counter: 0,
      listCounter: 1,
      tooltip: false,
      errorMessage: null,
    };
  },
  methods: {
    clearNumber() {
      this.places.pop();
      this.address = this.address.replace(`${this.number}, `, '')
      this.number = '';
      this.counter++;
    },
    showClear() {
      this.mouseOverField = true;
    },
    appendNumber() {
      if(this.number !== '') {
        this.places.push(`${this.number}, ${this.address}`);
        this.address = `${this.number}, ${this.address}`;
      } else {
        this.counter++;
      }
    },
    showTooltip(ev) {
      if(window.screen.orientation.type.includes('landscape') && ev === 'click') {
        this.tooltip = true;
      } else if(window.screen.orientation.type.includes('portrait') && ev === 'over') {
        this.tooltip = true;
      }
    },
    addForm() {
      this.showAdd = false;
      this.places = [];
      this.$emit('addForm');
      this.listCounter++;
    },
    call() {
      // To prevent CORS issues, proxy is used for local testing;
      // Be sure to enter your google places, API key in nuxt.config.js/env.API_KEY
      const proxy = process.env.NODE_ENV === 'development' ? 'https://cors-anywhere.herokuapp.com/' : '';
      this.$axios.get(`${proxy}https://maps.googleapis.com/maps/api/place/autocomplete/json?input=${this.address}&key=${process.env.API_KEY}&libraries=places`)
      .then((resp) => {
        console.log(resp);
        if(resp.data.error_message) {
          this.errorMessage = resp.data.error_message;
        } else {
          this.places = resp.data.predictions;
        }
      });
    },
  },
  watch: {
    address() {
      if(this.address !== '') {
        this.showAdd = true;
      } else {
        this.showAdd = false;
      }
    },
  },
  mounted() {
    console.log(window.screen.orientation.type);
  },
};
</script>

<style lang="scss">

.form-container {
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
  flex-flow: column;
  width: 100%;
  align-items: center;
  border-bottom: solid 1px rgba(255,255,255,0.4);
  padding-top:50px;

  @include mqBreakPoint(1023px) {
    flex-flow: row;
  }

  .select-container {
    width: 100%;
    position: relative;
    padding: 0 20px;
    margin-bottom: 50px;

    input[type="text"] {
      width: 100%;
      border-bottom: 1px solid white;
      -moz-appearance: none;
      -webkit-appearance: none;
      appearance: none;
      padding-bottom: 5px;
      line-height: 0.1px;
      &:focus {
        outline: none black 1px !important;
      }
      &.prediction {
        @include mqBreakPoint(1023px) {
          width: 500px;
        }
      }
    }

    .icon {
      position: absolute;
      transform:rotate(-90deg);
      right: 20px;
      top: 0;
      pointer-events: none;
      z-index: 100;

      &.clear {
        cursor: pointer;
        pointer-events: all;
      }

      &.tt {
        left: -10px;
        right: auto;
        transform:rotate(0deg);
        pointer-events: all;
        cursor:pointer;
        top: auto;
        bottom: -10px;
      }

      &.add-form {
        top: auto;
        pointer-events: all;
        bottom: -30px;
        cursor: pointer;
      }
    }
    @include mqBreakPoint(1023px) {
      width: fit-content;
    }

    .select {
      -moz-appearance: none;
      -webkit-appearance: none;
      appearance: none;
      padding-right: 30px;
      color: white;
      width: 100%;
      &:focus {
        outline: none black 1px !important;
      }

      &.gray {
        opacity: 0.6;
      }

      @include mqBreakPoint(1023px) {
        width: fit-content;
      }
    }

    .tooltip {
      position: absolute;
      bottom: -30px;
      left: 0;
      .icon {
        position: relative;

        &.clear {
          cursor: pointer;
          pointer-events: all;
          right: 0;
          top: 10px;
        }
      }
    }

    label {
      position: absolute;
      left: 50%;
      transform: translateX(-50%);
      bottom: -20px;
      text-transform: uppercase;
      }
    }
  }
</style>
