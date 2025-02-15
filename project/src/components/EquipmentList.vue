<template>
  <v-container
    ref="container"
    style="height: 100%; display: flex; flex-flow: column; width: 100%"
  >
    <v-sheet
      tile
      outlined
      style="width: 100%; flex: 1 1 auto"
      class="pa-0 ma-0 pb-1"
      elevation="1"
      rounded
    >
      <v-row no-gutters class="ml-2 align-baseline" style="width: 100%">
        <v-col cols="3"
          ><v-checkbox
            v-model="isLevel99"
            label="Lv99"
            dense
            hide-details
            class="pl-2 pr-2"
          ></v-checkbox
        ></v-col>
        <v-col cols="3"
          ><v-checkbox
            v-show="slotHasItemLevel"
            v-model="isItemLevel119"
            label="IL119"
            dense
            hide-details
          ></v-checkbox
        ></v-col>
        <v-col cols="6">
          <v-text-field
            :validate-on-blur="true"
            class="pt-3 pl-2 pr-6"
            clearable
            :label="$t('ui.filter.searchBoxLabel')"
            v-model="nameFilter"
            @click:clear="nameFilterCleared"
            outlined
            hide-details
            dense
            height="10px"
          ></v-text-field
        ></v-col>
      </v-row>
      <v-row no-gutters class="ml-2" style="width: 100%">
        <v-col class="pl-2"
          ><v-checkbox
            v-model="isCalUnity"
            :label="$t('ui.item.calculateunitycheck')"
            dense
            hide-details
          ></v-checkbox
        ></v-col>
        <v-col class="pl-2"
          ><v-checkbox
            v-model="isCalMaxAug"
            :label="$t('ui.item.calculatemaxaug')"
            dense
            hide-details
          ></v-checkbox
        ></v-col>
      </v-row>

      <v-row no-gutters class="pl-4 mt-1 pr-0 pb-0 mb-0 align-center">
        <v-col cols="3"
          ><v-btn
            v-on:click="addLimiter"
            class="pr-3"
            dense
            small
            elevation="1"
            color="blue lighten-3"
            >{{ $t("ui.filter.addPropFilter") }}</v-btn
          ></v-col
        >
        <v-col cols="3">
          <v-btn
            v-on:click="addSorter"
            class="mr-0 pr-4"
            dense
            small
            elevation="1"
            color="orange lighten-3"
            >{{ $t("ui.filter.addSort") }}</v-btn
          ></v-col
        >
        <v-col cols="6" class="pr-4" v-if="getTypeList() != undefined">
          <v-select
            solo
            :items="getTypeList()"
            v-model="selectedType"
            filled
            dense
            flat
            hide-details
            outlined
            :label="$t('ui.item.type')"
            clearable
          ></v-select
        ></v-col>
      </v-row>
      <equipment-property-limit-unit
        class="pl-1 pr-3 pt-1 pb-1 ma-1"
        :limiters="limiters"
        :props-array="propsArray"
        @delete="deleteLimiter"
        @valueChanged="limiterChanged"
        @activeChanged="limiterActiveChanged"
        @sortOrderChanged="sortOrderChanged"
      />
    </v-sheet>
    <v-list-item-group
      class="fill-height"
      style="width: 100%; min-height: 100px; flex: 0 1 auto"
    >
      <DynamicScroller
        :items="equipData"
        :min-item-size="54"
        style="height: 100%"
        key-field="Id"
        ref="itemlist"
        class="scrollbarTheme"
      >
        <template v-slot="{ item, index, active }">
          <DynamicScrollerItem
            :item="item"
            :active="active"
            :size-dependencies="[
              item.JpDescription,
              item.EnDescription,
              item.AugCape,
              item.AugFixed,
              item.AugRoute,
            ]"
            :data-index="index"
          >
            <equipment-list-item
              :index="item.Id"
              :source="item"
              :limiters="limiters"
              :propDict="propDict"
              :cateDict="cateDict"
            >
            </equipment-list-item>
          </DynamicScrollerItem>
        </template>
      </DynamicScroller>
      <!--      <virtual-list-->
      <!--        style="overflow-y: auto"-->
      <!--        class="fill-height"-->
      <!--        ref="v-scroller"-->
      <!--        :data-key="'Id'"-->
      <!--        :data-sources="equipData"-->
      <!--        :data-component="itemComponent"-->
      <!--        :extra-props="{-->
      <!--          limiters: limiters,-->
      <!--          propDict: propDict,-->
      <!--          cateDict: cateDict,-->
      <!--        }"-->
      <!--        :keeps="50"-->
      <!--        :estimate-size="55"-->
      <!--      ></virtual-list>-->
    </v-list-item-group>
  </v-container>
</template>

<script lang="ts">
import { Component, Mixins, Prop, Vue, Watch } from "vue-property-decorator";
import { Equipment, Limiter } from "@/@types/equip-set";
import xiUtils from "@/mixins/xiutils";
import EquipmentPropertyLimitUnit from "@/components/EquipmentPropertyLimitUnit.vue";
import TextHighlight from "vue-text-highlight";
import VirtualList from "vue-virtual-scroll-list";
import EquipmentListItem from "@/components/EquipmentListItem.vue";
import { debounce } from "lodash";

@Component({
  components: {
    EquipmentPropertyLimitUnit,
    TextHighlight,
    VirtualList,
    EquipmentListItem,
  },
  name: "EquipmentList",
})
export default class EquipmentList extends Mixins(xiUtils) {
  selectedItem = 1;
  limiters: Array<Limiter> = [];
  isLevel99 = true;
  isItemLevel119 = true;
  nameFilter: string | null = "";
  selectedType: string | null = null;
  isCalUnity = true;
  isCalMaxAug = true;

  itemComponent = EquipmentListItem;

  @Prop({ default: "" }) readonly equipSlot: string | undefined;

  @Prop({ default: null }) readonly equipQueryChain!: any;
  @Prop() readonly propsArray!: Record<string, unknown>[];
  @Prop() readonly propDict!: Map<string, string[]>;
  @Prop() readonly cateDict!: Map<string, string[]>;

  equipData: Equipment[] = [];

  nonILSlots: string[] = [
    "Cape",
    "Neck",
    "Waist",
    "LEarring",
    "REarring",
    "LRing",
    "RRing",
  ];

  public getTypeList() {
    if (this.equipSlot === "Main") {
      return this.getMainHandTypeList();
    }
    if (this.equipSlot === "Sub") {
      return this.getSubHandTypeList();
    }
    if (this.equipSlot === "Range") {
      return this.getRangeTypeList();
    }
    if (this.equipSlot === "Ammo") {
      return this.getAmmoTypeList();
    }
    return undefined;
  }

  get slotHasItemLevel(): boolean {
    return !(
      this.equipSlot != undefined && this.nonILSlots.includes(this.equipSlot)
    );
  }

  public nameFilterCleared(): void {
    this.nameFilter = "";
  }

  @Watch("equipSlot")
  slotChanged() {
    this.selectedType = null;
  }

  @Watch("nameFilter")
  queryChanged() {
    this.queryChangedDebounced();
  }

  @Watch("equipQueryChain")
  @Watch("isLevel99")
  @Watch("isItemLevel119")
  @Watch("limiters")
  @Watch("isCalUnity")
  @Watch("isCalMaxAug")
  @Watch("nameFilter")
  @Watch("selectedType")
  queryChangedInstant() {
    if (this.equipQueryChain != null) {
      let chain = this.equipQueryChain.copy();
      let query: Record<string, any> = {};

      if (this.isLevel99) {
        query["Level"] = 99;
      }
      if (this.isItemLevel119 && this.slotHasItemLevel) {
        query["ItemLevel"] = 119;
      }
      if (Object.keys(query).length !== 0) {
        chain = chain.find(query);
      }

      if (
        this.equipSlot === "Main" ||
        this.equipSlot === "Sub" ||
        this.equipSlot === "Range" ||
        this.equipSlot === "Ammo"
      ) {
        const selectedType = this.selectedType;
        if (this.selectedType != undefined && this.selectedType != "") {
          const getType = this.getItemTypeName;
          chain = chain.where(function (obj: Equipment) {
            const type = getType(obj);
            if (type === selectedType) {
              return true;
            }
            return false;
          });
        }
      }

      let nameFilter = this.nameFilter;
      if (nameFilter !== "" && nameFilter !== null) {
        nameFilter = this.kataToHira(
          this.fullWidthStrToHalfWidthStr(nameFilter as string).toLowerCase()
        );
        let funcKataHira = this.kataToHira;
        let funcFullTohalf = this.fullWidthStrToHalfWidthStr;
        let augStringGetter = this.getAugStringForSearch;
        let propDict = this.propDict;
        let cateDict = this.cateDict;
        let locale = this.$i18n.locale;
        chain = chain.where(function (obj: Equipment) {
          let isPassed = false;
          if (
            funcKataHira(funcFullTohalf(obj.Jp.toLowerCase())).includes(
              nameFilter as string
            )
          ) {
            isPassed = true;
          } else if (
            funcKataHira(funcFullTohalf(obj.JpFull.toLowerCase())).includes(
              nameFilter as string
            )
          ) {
            isPassed = true;
          } else if (obj.En.toLowerCase().includes(nameFilter as string)) {
            isPassed = true;
          } else if (obj.EnFull.toLowerCase().includes(nameFilter as string)) {
            isPassed = true;
          } else if (
            obj.JpDescription != null &&
            funcKataHira(funcFullTohalf(obj.JpDescription))
              .toLowerCase()
              .includes(nameFilter as string)
          ) {
            isPassed = true;
          } else if (
            obj.EnDescription != null &&
            funcKataHira(funcFullTohalf(obj.EnDescription))
              .toLowerCase()
              .includes(nameFilter as string)
          ) {
            isPassed = true;
          } else if (
            obj.AugCape != null &&
            funcKataHira(
              funcFullTohalf(augStringGetter(obj.AugCape, propDict, cateDict))
            )
              .toLowerCase()
              .includes(nameFilter as string)
          ) {
            isPassed = true;
          } else if (
            obj.AugFixed != null &&
            funcKataHira(
              funcFullTohalf(
                augStringGetter([obj.AugFixed], propDict, cateDict)
              )
            )
              .toLowerCase()
              .includes(nameFilter as string)
          ) {
            isPassed = true;
          } else if (
            obj.AugRoute != null &&
            funcKataHira(
              funcFullTohalf(augStringGetter(obj.AugRoute, propDict, cateDict))
            )
              .toLowerCase()
              .includes(nameFilter as string)
          ) {
            isPassed = true;
          }

          return isPassed;
        });
      }

      if (this.limiters.length !== 0) {
        for (let i = 0; i < this.limiters.length; i++) {
          if (!this.limiters[i].isActive) {
            continue;
          }

          if (this.limiters[i].isProp) {
            let testPropID = this.limiters[i].propertyID;
            let testCatID = this.limiters[i].categoryID;
            let calUnity = this.isCalUnity;
            if (testPropID == undefined) {
              continue;
            }
            let min = this.limiters[i].minValue;
            chain = chain.where(function (obj: Equipment) {
              let props = obj.Properties;
              let augsCape = obj.AugCape;
              let augsFix = obj.AugFixed;
              let augsRoute = obj.AugRoute;
              let totalValue = 0;
              let testPassed = false;
              if (
                props == undefined &&
                augsCape == undefined &&
                augsFix == undefined &&
                augsRoute == undefined
              ) {
                return false;
              }
              // match test
              if (props != undefined) {
                for (let prop of props) {
                  if (testPropID == null) {
                    return false;
                  }
                  if (testPropID === prop.PropID) {
                    if (
                      (testCatID == null && prop.CatID == null) ||
                      testCatID === prop.CatID
                    ) {
                      totalValue += Math.abs(prop.Value ?? 0);
                      testPassed = true;
                    } else if (calUnity) {
                      if (testCatID == null && prop.CatID == 13) {
                        totalValue += Math.abs(prop.Value ?? 0);
                        testPassed = true;
                      } else if (testCatID == 8 && prop.CatID == 195) {
                        totalValue += Math.abs(prop.Value ?? 0);
                        testPassed = true;
                      }
                    }
                  }
                }
              }

              // Calculate All Routes Total
              if (augsCape != undefined) {
                for (const augs of augsCape) {
                  for (const aug of augs.Augs) {
                    for (const augProp of aug.PropIDs) {
                      if (
                        testPropID == augProp &&
                        ((testCatID == null && aug.CatID == null) ||
                          testCatID == aug.CatID)
                      ) {
                        totalValue += Math.abs(aug.Value ?? 0);
                        testPassed = true;
                        break;
                      }
                    }
                    if (testPassed) break;
                  }
                  if (testPassed) break;
                }
              }

              // Calculate All Routes Total
              if (augsFix != undefined) {
                for (const aug of augsFix.Augs) {
                  for (const augProp of aug.PropIDs) {
                    if (
                      testPropID == augProp &&
                      ((testCatID == null && aug.CatID == null) ||
                        testCatID == aug.CatID)
                    ) {
                      totalValue += Math.abs(aug.Value ?? 0);
                      testPassed = true;
                      break;
                    }
                  }
                  if (testPassed) break;
                }
              }

              // Calculate Max Route Value
              let routeMax = 0;
              if (augsRoute != undefined) {
                for (const augs of augsRoute) {
                  for (const aug of augs.Augs) {
                    for (const augProp of aug.PropIDs) {
                      if (
                        testPropID == augProp &&
                        ((testCatID == null && aug.CatID == null) ||
                          testCatID == aug.CatID)
                      ) {
                        const value = Math.abs(aug.Value ?? 0);
                        if (value >= routeMax) {
                          testPassed = true;
                          routeMax = value;
                        }
                        break;
                      }
                    }
                  }
                }
              }
              totalValue += routeMax;
              return testPassed && totalValue >= min;
            });
          }

          // Sorting Chains
          if (this.limiters[i].isSort) {
            let funcGetValue = this.getPropertyValue;
            let isAsc = this.limiters[i].isAsc;
            const filterPropID = this.limiters[i].propertyID;
            const filterCateID = this.limiters[i].categoryID;
            if (filterPropID == undefined) {
              continue;
            }
            const calUnity = this.isCalUnity;
            const calAug = this.isCalMaxAug;
            chain = chain.sort(function (obj1: Equipment, obj2: Equipment) {
              const value1 = funcGetValue(
                obj1,
                filterPropID,
                filterCateID,
                calUnity,
                calAug
              );
              const value2 = funcGetValue(
                obj2,
                filterPropID,
                filterCateID,
                calUnity,
                calAug
              );
              let result = 0;
              if (value1 === undefined && value2 === undefined) {
                result = 0;
              }

              if (value1 === undefined && value2 !== undefined) {
                result = -1;
              }

              if (value1 !== undefined && value2 === undefined) {
                result = 1;
              }

              if ((value1 as number) > (value2 as number)) {
                result = 1;
              }
              if ((value1 as number) < (value2 as number)) {
                result = -1;
              }
              if ((value1 as number) === (value2 as number)) {
                result = 0;
              }

              if (!isAsc) {
                result = -1 * result;
              }
              return result;
            });
          }
        }
      }

      this.equipData = Object.freeze(chain.data());
      this.scrollToTop();
    }
  }

  queryChangedDebounced() {
    this.queryChangedInstant();
  }

  public addLimiter(): void {
    this.limiters.push({
      index: Date.now(),
      propertyID: null,
      categoryID: null,
      text: null,
      minValue: 0,
      isActive: true,
      isSort: false,
      isText: false,
      isProp: true,
      isAsc: false,
    });
  }

  // public addTextLimiter(): void {
  //   this.limiters.push({
  //     index: Date.now(),
  //     propertyID: undefined,
  //     categoryID: undefined,
  //     text: null,
  //     minValue: 0,
  //     isActive: true,
  //     isSort: false,
  //     isText: true,
  //     isProp: false,
  //     isAsc: false,
  //   });
  // }

  public addSorter(): void {
    for (const lim of this.limiters) {
      if (lim.isSort) {
        return;
      }
    }
    this.limiters.push({
      index: Date.now(),
      propertyID: null,
      categoryID: null,
      text: null,
      minValue: 0,
      isActive: true,
      isSort: true,
      isText: false,
      isProp: false,
      isAsc: false,
    });
  }

  public deleteLimiter(index: number): void {
    for (let i = 0; i < this.limiters.length; i++) {
      if (this.limiters[i].index == index) {
        this.limiters.splice(i, 1);
      }
    }
  }

  public limiterChanged(obj: Record<string, any>): void {
    if (obj["index"] != undefined) {
      for (let limiter of this.limiters) {
        if (limiter.index == obj["index"]) {
          if (obj["text"] != undefined) {
            limiter.text = obj["text"].replace("：", ":");
          }
          if (obj["property"] != undefined) {
            limiter.propertyID = obj["property"][0];
            limiter.categoryID = obj["property"][1];
          }

          //console.log("propID" + limiter.propertyID);

          if (obj["minValue"] != undefined) {
            limiter.minValue = obj["minValue"];
          }
          this.queryChanged();
        }
      }
    }
  }

  public limiterActiveChanged(obj: Record<string, any>): void {
    if (obj["index"] != undefined) {
      for (let limiter of this.limiters) {
        if (limiter.index == obj["index"]) {
          limiter.isActive = obj["isActive"];
          this.queryChanged();
        }
      }
    }
  }

  public sortOrderChanged(obj: Record<string, any>): void {
    if (obj["index"] != undefined) {
      for (let limiter of this.limiters) {
        if (limiter.index == obj["index"]) {
          limiter.isAsc = obj["isAsc"];
          this.queryChanged();
        }
      }
    }
  }

  public scrollToTop() {
    ((this.$refs.itemlist as Vue).$el as HTMLDivElement).scrollTo({
      top: 0,
      left: 0,
      behavior: "auto",
    });
  }

  created() {
    this.queryChangedDebounced = debounce(this.queryChangedDebounced, 500);
  }
}
</script>

<style scoped>
.scrollbarTheme::-webkit-scrollbar {
  width: 15px;
}

.scrollbarTheme::-webkit-scrollbar-track {
  background: #eaeaea;
  border-left: 1px solid #dadada;
}

.scrollbarTheme::-webkit-scrollbar-thumb {
  background: #b0b0b0;
  border: solid 3px #e6e6e6;
  border-radius: 7px;
}

.scrollbarTheme::-webkit-scrollbar-thumb:hover {
  background: black;
}
</style>
