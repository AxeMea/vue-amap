<template>
  <div class="el-amap-city-selector">
    <label for="" class="el-amap-city-selector__label" v-if="showProvince">省：</label>

    <select
      class="el-amap-city-selector__select"
      v-if="showProvince"
      :disabled="provinceDisabled"
      @change="provinceChangeHandler($event.target.value)">
      <option
        v-for="option in provinceOptions"
        :value="option.adcode"
      >{{ option.name }}</option>
    </select>

    <label for="" class="el-amap-city-selector__label" v-if="showCity">市：</label>

    <select
      class="el-amap-city-selector__select"
      v-if="showCity"
      :disabled="cityDisabled"
      @change="cityChangeHandler($event.target.value)">
      <option
        v-for="option in cityOptions"
        :value="option.adcode"
      >{{ option.name }}</option>
    </select>

    <label for="" class="el-amap-city-selector__label" v-if="showArea">区：</label>

    <select
      class="el-amap-city-selector__select"
      v-if="showArea"
      :disabled="areaDisabled"
      @change="areaChangeHandler($event.target.value)">
      <option
        v-for="option in areaOptions"
        :value="option.adcode"
      >{{ option.name }}</option>
    </select>
  </div>
</template>
<script>
import { lazyAMapApiLoaderInstance } from '../services/injected-amap-api-instance';

export default {
  name: 'el-amap-city-selector',

  props: {
    showProvince: {
      type: Boolean,
      default: true
    },
    provinceDisabled: {
      type: Boolean,
      default: false
    },
    province: {
      type: String,
      default: ''
    },
    showCity: {
      type: Boolean,
      default: true
    },
    cityDisabled: {
      type: Boolean,
      default: false
    },
    city: {
      type: String,
      default: ''
    },
    showArea: {
      type: Boolean,
      default: true
    },
    areaDisabled: {
      type: Boolean,
      default: false
    },
    area: {
      type: String,
      default: ''
    }
  },

  data() {
    return {
      provinceValue: '',
      provinceOptions: [],
      cityValue: '',
      cityOptions: [],
      areaValue: '',
      areaOptions: [],
      districtPlugin: '',
      districtCache: {
        country: {},
        province: {},
        city: {}
      },
      defaultOption: {
        adcode: '',
        name: '请选择'
      }
    };
  },

  methods: {
    loadPlugin() {
      if (this.districtPlugin) {
        return Promise.resolve(this.districtPlugin);
      }

      return new Promise((resolve, reject) => {
        lazyAMapApiLoaderInstance.load().then(() => {
          this.districtPlugin = new AMap.DistrictSearch({
            subdistrict: 1
          });
          return resolve(this.districtPlugin);
        }).catch(reject);
      });
    },

    fetchProvince() {
      this.search('country', '中国')
        .then(districts => {
          this.provinceOptions = [
            this.defaultOption
          ].concat(districts);

          this.resetCity();
          this.resetArea();
        });
    },

    fetchCity(parentCode) {
      this.search('province', parentCode)
        .then(districts => {
          this.cityOptions = [
            this.defaultOption
          ].concat(districts);
        });
    },

    fetchArea(parentCode) {
      this.search('city', parentCode)
        .then(districts => {
          this.areaOptions = [
            this.defaultOption
          ].concat(districts);
        });
    },

    keyGenerator(district) {
      let key = '';

      if (district.adcode !== undefined) {
        key = district.adcode;
      } else if (district.citycode !== undefined) {
        key = district.citycode;
      } else if (district.name && district.level) {
        key = `${ district.name }_${ district.level }`;
      }

      return key;
    },

    search(level, keyword) {
      return new Promise((resolve, reject) => {
        this.loadPlugin().then(plugin => {
          plugin.setLevel(level);

          plugin.search(keyword, (status, result) => {
            let districts = [];
            let key = '';

            if (status === 'complete' && result && result.districtList) {
              let districtData = result.districtList[0];
              districts = districtData.districtList || districtData.districts;

              if (districts && districts.length > 0) {
                districts.forEach(dist => {
                  key = this.keyGenerator(dist);
                  this.districtCache[level][key + ''] = dist;
                });
              }
            }
            resolve(districts);
          });
        });
      });
    },

    resetCity() {
      this.cityValue = '';
      this.cityOptions = [this.defaultOption];
    },

    resetArea() {
      this.areaValue = '';
      this.areaOptions = [this.defaultOption];
    },

    getCache(level, key) {
      return JSON.parse(JSON.stringify(this.districtCache[level][key + '']));
    },

    provinceChangeHandler(value) {
      if (value) {
        this.resetCity();
        this.resetArea();

        this.fetchCity(value);

        let meta = this.getCache('country', value);
        this.$emit('province-change', meta);
      }
    },

    cityChangeHandler(value) {
      if (value) {
        this.resetArea();

        this.fetchArea(value);
        let meta = this.getCache('province', value);
        this.$emit('city-change', meta);
      }
    },

    areaChangeHandler(value) {
      if (value) {
        let meta = this.getCache('city', value);
        this.$emit('area-change', meta);
      }
    }
  },

  mounted() {
    this.fetchProvince().then();

    if (this.propsData.province) {

    }
  }
};
</script>

<style lang="scss">
.el-amap-city-selector {

  &__label {
    margin: 0 10px;

    &:first-child {
      margin-left: 0;
    }
  }

  &__select {
    width: 110px;
    height: 33px;
    border: 1px #d4d4d4 solid;
    border-radius: 0;
    appearance: none;
    -moz-appearance: none;
    -webkit-appearance: none;
    padding: 5px 15px 5px 10px;
    border-radius: 4px;
    background-color: #fcfcfc;
  }
}
</style>
