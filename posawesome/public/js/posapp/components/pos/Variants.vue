<template>
  <v-row justify="center">
    <v-dialog v-model="varaintsDialog" max-width="600px">
      <v-card min-height="500px">
        <v-card-title>
          <span class="headline primary--text">إختر نوع الخدمة</span>
          <v-spacer></v-spacer>
          <v-btn color="error" dark @click="close_dialog">أغلق</v-btn>
        </v-card-title>
        <v-card-text class="pa-0">
          <v-container v-if="parentItem">
            <!-- aswh edit : to hide varaints filters-->
            <!-- <div v-for="attr in parentItem.attributes" :key="attr.attribute">
              <v-chip-group
                v-model="filters[attr.attribute]"
                active-class="green--text text--accent-4"
                column
              >
                <v-chip
                  v-for="value in attr.values"
                  :key="value.abbr"
                  :value="value.attribute_value"
                  outlined
                  label
                  @click="updateFiltredItems"
                >
                  {{ value.attribute_value }}
                </v-chip>
              </v-chip-group>
              <v-divider class="p-0 m-0"></v-divider>
            </div> -->
            <div>
              <v-row dense class="overflow-y-auto" style="max-height: 500px">
                <v-col
                  v-for="(item, idx) in filterdItems"
                  :key="idx"
                  xl="2"
                  lg="3"
                  md="4"
                  sm="4"
                  cols="6"
                  min-height="50"
                >
                  <v-card hover="hover" @click="add_item(item)">
                   <!-- aswh_edit () -->

                    <v-img v-if="item.item_name.includes('غ')"
                      :src="'/assets/posawesome/js/posapp/components/pos/aswh_edit_clean.png'"
                      class="white--text align-end"
                      style="border: 1px solid rgb(151, 151, 151);"
                      height="100px"
                    >
                    </v-img>
                    <v-img v-else
                      :src="'/assets/posawesome/js/posapp/components/pos/aswh_edit_iron.png'"
                      class="white--text align-end"
                      style="border: 1px solid rgb(151, 151, 151);"
                      height="100px"
                    >
                    </v-img>
                    <!-- <v-card-text class="text--primary pa-1">
                      <div class="text-caption primary--text accent-3">
                        {{ item.rate || 0 }} {{ item.currency || '' }}
                      </div>
                    </v-card-text> -->
                  </v-card>
                </v-col>
              </v-row>
            </div>
          </v-container>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
import { evntBus } from '../../bus';
export default {
  data: () => ({
    varaintsDialog: false,
    parentItem: null,
    items: null,
    filters: {},
    filterdItems: [],
  }),

  computed: {
    variantsItems() {
      if (!this.parentItem) {
        return [];
      } else {
        return this.items.filter(
          (item) => item.variant_of == this.parentItem.item_code
        );
      }
    },
  },

  methods: {
    close_dialog() {
      this.varaintsDialog = false;
    },
    formtCurrency(value) {
      value = parseFloat(value);
      return value.toFixed(2).replace(/\d(?=(\d{3})+\.)/g, '$&,');
    },
    updateFiltredItems() {
      this.$nextTick(function () {
        const values = [];
        Object.entries(this.filters).forEach(([key, value]) => {
          if (value) {
            values.push(value);
          }
        });

        if (!values.length) {
          this.filterdItems = this.variantsItems;
        } else {
          const itemsList = [];
          this.filterdItems = [];
          this.variantsItems.forEach((item) => {
            let apply = true;
            item.item_attributes.forEach((attr) => {
              if (
                this.filters[attr.attribute] &&
                this.filters[attr.attribute] != attr.attribute_value
              ) {
                apply = false;
              }
            });
            if (apply && !itemsList.includes(item.item_code)) {
              this.filterdItems.push(item);
              itemsList.push(item.item_code);
            }
          });
        }
      });
    },
    add_item(item) {
      evntBus.$emit('add_item', item);
      // aswh_edit : to disable varaintsDialog closing after clicking
      // this.close_dialog();
    },
  },

  created: function () {
    evntBus.$on('open_variants_model', (item, items) => {
      this.varaintsDialog = true;
      this.parentItem = item || null;
      this.items = items;
      this.filters = {};
      this.$nextTick(function () {
        this.filterdItems = this.variantsItems;
      });
    });
  },
};
</script>
