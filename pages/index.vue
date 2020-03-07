<template>
  <v-layout column style="position: relative;">
    <v-row>
      <v-col md="6">
        <div id="map-wrap" style="height: 100%;">
          <client-only>
            <l-map ref="map" :zoom="zoom" :center="center">
              <l-tile-layer
                :attribution="attribution"
                url="https://{s}.tile.osm.org/{z}/{x}/{y}.png"
              />
            </l-map>
          </client-only>
        </div>
      </v-col>
      <v-col md="6">
        <v-expansion-panels v-model="currentItem">
          <v-expansion-panel v-for="item in items" :key="item.name">
            <v-expansion-panel-header>{{ item.name }}</v-expansion-panel-header>
            <v-expansion-panel-content>
              <v-container>
                <v-row>
                  <v-col md="8" col="12">
                    <h3>概要</h3>
                    <p>{{ item.description }}</p>

                    <p v-if="item.hp.title">
                      HP: <a :href="item.hp.url">{{ item.hp.title }}</a>
                    </p>
                  </v-col>
                  <v-col md="4" col="12" class="text-center">
                    <img
                      :src="
                        item.image_yatai ? item.image_yatai : 'no-image.svg'
                      "
                      class="yatai_img"
                      alt="屋台イメージ"
                    />
                  </v-col>
                </v-row>
              </v-container>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </v-expansion-panels>
      </v-col>
    </v-row>
  </v-layout>
</template>

<script>
/* globals L */

export default {
  data() {
    return {
      center: [34.747445, 134.800173],
      zoom: 15,
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      items: [],
      currentItem: false
    }
  },

  watch: {
    currentItem() {
      const item = this.items[this.currentItem]
      if (item && this.$refs.map.mapObject) {
        this.$refs.map.mapObject.panTo([item.location[1], item.location[0]])
        this.layers.yatai.eachLayer((layer) => {
          if (layer.feature && layer.feature.properties.name === item.name) {
            layer.openPopup()
          } else {
            layer.closePopup()
          }
        })
      } else {
        this.layers.yatai.eachLayer((layer) => {
          layer.closePopup()
        })
      }
    }
  },

  mounted() {
    setTimeout(() => {
      this.layers = {}
      this.initLayers(this.$refs.map.mapObject)
    }, 100)
  },

  methods: {
    initLayers(map) {
      this.$axios
        .get('/data.json')
        .then((res) => {
          this.items = res.data
          const geojson = {
            type: 'FeatureCollecton',
            features: res.data.map((d) => {
              const properties = Object.assign({}, d)
              delete properties.location
              return {
                type: 'Feature',
                geometry: {
                  type: 'Point',
                  coordinates: d.location
                },
                properties
              }
            })
          }
          this.layers.yatai = L.geoJSON(geojson)
            .eachLayer((layer) => {
              const props = layer.feature.properties
              const img = `<img class="popup_image" src="${
                props.image_kura ? props.image_kura : 'no-image.svg'
              }" alt="屋台蔵イメージ">`
              layer.bindPopup(`<div><p>${props.name}</p>${img}</div>`)
            })
            .on('click', (e) => {
              if (e.layer && e.layer.feature && e.layer.feature.properties) {
                this.currentItem = this.items.findIndex(
                  (i) => i.name === e.layer.feature.properties.name
                )
              }
            })
            .addTo(map)
        })
        .catch(() => {
          alert('データの取得に失敗しました')
        })
    }
  }
}
</script>

<style lang="scss">
.yatai_img {
  max-height: 200px;
}

.popup_image {
  height: 150px;
  width: 150px;
}
</style>
