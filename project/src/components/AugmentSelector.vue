<template>
  <v-card
    style="width: 100%; height: 100%"
    :class="'ma-0 pa-0 '"
    outlined
    flat
    elevation="0"
  >
    <v-card-title class="pa-0 pl-3 blue-grey darken-1 white--text">
      <v-icon left color="white"> mdi-playlist-plus </v-icon>
      <span>{{ $t("ui.item.propertyModification") }}</span>
    </v-card-title>
    <v-alert outlined dense type="warning" border="left" class="ma-2 mb-0">
      {{ $t("ui.item.disclaimerAug") }}
    </v-alert>
    <v-row no-gutters>
      <v-col cols="12" class="pt-2 pl-2 pr-1">
        <v-row no-gutters class="flex-row mb-1">
          <v-col cols="2">
            <v-card
              color="blue-grey lighten-4"
              outlined
              flat
              elevation="0"
              width="100"
            >
              <v-card-title
                class="
                  blue-grey
                  lighten-1
                  pa-0
                  flex-row
                  text-center
                  align-center
                  justify-center
                "
              >
                <span
                  class="
                    align-self-center
                    text-center
                    white--text
                    pa-0
                    text-body-2
                  "
                  >{{ labelName }}</span
                >
              </v-card-title>
              <v-sheet
                color="blue-grey lighten-4"
                class="justify-center align-center flex-row"
                height="40px"
              >
                <v-img
                  aspect-ratio="1"
                  contain
                  :src="iconUrl"
                  height="32px"
                  class="mt-1"
                ></v-img>
              </v-sheet>
              <v-sheet
                style="width: 100%"
                class="
                  blue-grey
                  lighten-5
                  pa-0
                  text-center
                  justify-center
                  align-self-baseline
                "
              >
                <p
                  style="
                    width: 100%;
                    overflow-wrap: normal;
                    text-overflow: ellipsis;
                  "
                  class="
                    text-center
                    blue-grey--text
                    darken-3--text
                    pl-1
                    pr-1
                    pa-0
                    ma-0
                    text-caption
                  "
                >
                  {{ itemName }}
                </p>
              </v-sheet>
            </v-card>
          </v-col>
          <v-col cols="10">
            <v-card
              outlined
              flat
              elevation="0"
              class="ml-1 mr-1 flex-row"
              color="blue-grey lighten-5"
            >
              <v-card-title
                class="
                  blue-grey
                  lighten-1
                  pa-0
                  flex-row
                  text-center
                  align-center
                  justify-center
                "
              >
                <span
                  class="
                    align-self-center
                    text-center
                    white--text
                    pa-0
                    text-caption
                  "
                  >{{ $t("ui.item.properties") }}</span
                >
              </v-card-title>
              <v-row class="align-center pl-1 pt-1 pb-1" no-gutters>
                <v-chip
                  v-show="itemTypeName"
                  class="mr-1"
                  color="black"
                  label
                  text-color="white"
                  x-small
                >
                  {{ itemTypeName }}
                </v-chip>
                <v-chip
                  v-if="isRare"
                  class="mr-1"
                  color="yellow darken-3"
                  label
                  text-color="white"
                  x-small
                  >Rare
                </v-chip>
                <v-chip
                  v-if="isEx"
                  class="mr-1"
                  color="green"
                  label
                  text-color="white"
                  x-small
                  >Ex
                </v-chip>
                <v-chip
                  class="mr-1"
                  color="pink"
                  label
                  text-color="white"
                  x-small
                  >{{ "Lv." + source.Level }}
                </v-chip>
                <v-chip
                  v-if="source.ItemLevel !== undefined"
                  class="mr-1"
                  color="purple"
                  label
                  text-color="white"
                  x-small
                  >{{ "IL." + source.ItemLevel }}
                </v-chip>
                <v-chip
                  v-if="source.SuLevel !== undefined"
                  class="mr-1"
                  color="blue"
                  label
                  text-color="white"
                  x-small
                  >{{ "Su" + source.SuLevel }}
                </v-chip>
              </v-row>
              <v-row
                no-gutters
                style="font-size: 7px"
                class="
                  blue-grey--text
                  lighten-5--text
                  text-body-2
                  pb-1
                  pt-1
                  pl-1
                "
                >{{ jobsString }}</v-row
              >
              <v-row no-gutters class="pl-1">
                <span
                  class="grey--text text--darken-1"
                  style="
                    word-break: keep-all;
                    overflow-wrap: break-word;
                    line-height: 1.4em;
                  "
                  >{{ itemDescription }}
                </span>
              </v-row>
              <v-row no-gutters class="mb-2 pl-1">
                <v-chip
                  :href="
                    'http://wiki.ffo.jp/search.cgi?CCC=%E6%84%9B&Command=Search&qf=' +
                    encodeURI(source.Jp.replace('+', '&2b')) +
                    '&order=match&ffotype=title&type=title'
                  "
                  class="mr-1"
                  color="orange lighten-1"
                  label
                  outlined
                  target="_blank"
                  x-small
                >
                  <v-icon left x-small> mdi-link-variant</v-icon>
                  FFO
                </v-chip>
                <v-chip
                  :href="
                    'https://www.bg-wiki.com/ffxi/' +
                    source.En.replace('\'', '.').replace(' ', '_')
                  "
                  class="mr-1"
                  color="brown lighten-1"
                  label
                  outlined
                  target="_blank"
                  x-small
                >
                  <v-icon left x-small> mdi-link-variant</v-icon>
                  BG
                </v-chip>
                <v-chip
                  :href="'https://www.ffxiah.com/item/' + source.Id"
                  class="mr-1"
                  color="blue lighten-1"
                  label
                  outlined
                  target="_blank"
                  x-small
                >
                  <v-icon left x-small> mdi-link-variant</v-icon>
                  AH
                </v-chip>
              </v-row>
            </v-card>
            <v-card
              v-if="calculatedAugs.length > 0"
              outlined
              flat
              elevation="0"
              class="mt-2 ml-1 mr-1 flex-row"
              color="blue-grey lighten-5"
            >
              <v-card-title
                class="
                  blue-grey
                  lighten-1
                  pa-0
                  flex-row
                  text-center
                  align-center
                  justify-center
                "
              >
                <span
                  class="
                    align-self-center
                    text-center
                    white--text
                    pa-0
                    text-caption
                  "
                  >{{ $t("ui.item.augments") }}</span
                >
              </v-card-title>
              <v-row no-gutters class="pl-1 mt-1 mb-1"
                ><span
                  class="grey--text text--darken-1"
                  style="word-break: break-word; line-height: 1.4em"
                  >{{ calculatedAugsString }}
                </span></v-row
              >
            </v-card>
          </v-col>
        </v-row>
      </v-col>

      <v-col cols="12" class="pa-1">
        <!-- Aug Capes -->
        <v-row no-gutters v-if="augCapes != null" class="ml-1 mr-1">
          <v-card
            color="red lighten-4"
            outlined
            flat
            elevation="0"
            width="100%"
          >
            <v-card-title
              class="
                red
                lighten-2
                pa-0
                flex-row
                text-center
                align-center
                justify-center
              "
            >
              <span
                class="
                  align-self-center
                  text-center
                  white--text
                  pa-0
                  text-caption
                "
                >{{ $t("ui.item.addAugments") }}</span
              >
            </v-card-title>
            <v-row
              no-gutters
              v-for="(augSet, index) in augCapes"
              v-bind:key="index"
              class="justify-center align-baseline"
            >
              <v-card outlined flat style="width: 100%">
                <v-row no-gutters>
                  <v-col cols="9" style="width: 100%">
                    <v-select
                      style="width: 100%"
                      :items="augSet.Augs"
                      :label="$t('ui.aug.' + augSet.Label)"
                      v-model="augProps[index]"
                      @change="augChanged($event, index)"
                      filled
                      flat
                      dense
                      disable-lookup
                      clearable
                      single-line
                      hide-details
                      background-color="deep-orange lighten-5"
                      color="grey"
                    >
                      <template slot="selection" slot-scope="data">
                        {{ getPropertyName(data.item) }}
                      </template>
                      <template slot="item" slot-scope="data">
                        {{ getPropertyName(data.item) }}
                      </template>
                    </v-select>
                  </v-col>
                  <v-col cols="3" class="">
                    <v-text-field
                      :label="$t('ui.item.value')"
                      background-color="grey lighten-5"
                      color="grey"
                      single-line
                      flat
                      hide-details
                      clearable
                      dense
                      filled
                      v-model="augValues[index]"
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-card>
            </v-row>
          </v-card>
        </v-row>
        <!-- AugFixed -->
        <v-row no-gutters v-if="augFixed != null" class="ml-1 mr-1">
          <v-card
            color="red lighten-4"
            outlined
            flat
            elevation="0"
            width="100%"
          >
            <v-card-title
              class="
                red
                lighten-2
                pa-0
                flex-row
                text-center
                align-center
                justify-center
              "
            >
              <span
                class="
                  align-self-center
                  text-center
                  white--text
                  pa-0
                  text-caption
                "
                >{{ $t("ui.item.addAugments") }}</span
              >
            </v-card-title>
            <v-row
              no-gutters
              v-for="(augProp, index) in augFixed.Augs"
              v-bind:key="index"
              class="justify-center align-baseline"
            >
              <v-card outlined flat style="width: 100%">
                <v-row no-gutters>
                  <v-col cols="1 pl-2">
                    <v-checkbox
                      flat
                      dense
                      :false-value="false"
                      :true-value="true"
                      hide-details
                      v-model="augToggles[index]"
                    ></v-checkbox>
                  </v-col>
                  <v-col cols="8" style="width: 100%">
                    <v-text-field
                      style="width: 100%"
                      v-if="augProp != null"
                      :value="getPropertyName(augProp)"
                      filled
                      flat
                      dense
                      disabled
                      disable-lookup
                      single-line
                      hide-details
                      background-color="deep-orange lighten-5"
                      color="grey"
                    >
                    </v-text-field>
                  </v-col>
                  <v-col cols="3" class="">
                    <v-text-field
                      :label="$t('ui.item.value')"
                      background-color="grey lighten-5"
                      color="grey"
                      single-line
                      flat
                      hide-details
                      clearable
                      dense
                      filled
                      v-model="augValues[index]"
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-card>
            </v-row>
          </v-card>
        </v-row>
        <!-- AugRoute -->
        <v-row no-gutters v-if="augRoute != null" class="ml-1 mr-1">
          <v-card
            color="red lighten-4"
            outlined
            flat
            elevation="0"
            width="100%"
          >
            <v-card-title
              class="
                red
                lighten-2
                pa-0
                flex-row
                text-center
                align-center
                justify-center
              "
            >
              <span
                class="
                  align-self-center
                  text-center
                  white--text
                  pa-0
                  text-caption
                "
                >{{ $t("ui.item.addAugments") }}</span
              >
            </v-card-title>
            <v-row no-gutters>
              <v-select
                style="width: 100%"
                v-if="augRoute != null"
                :items="augRoute"
                :value="augRoute[0]"
                @change="routeChanged($event)"
                filled
                flat
                dense
                disable-lookup
                single-line
                hide-details
                background-color="deep-orange lighten-5"
                color="grey"
              >
                <template slot="selection" slot-scope="data">
                  {{ $t("ui.aug." + data.item.Label) }}
                </template>
                <template slot="item" slot-scope="data">
                  {{ $t("ui.aug." + data.item.Label) }}
                </template>
              </v-select>
            </v-row>
            <v-row
              no-gutters
              v-for="(augProp, index) in augProps"
              v-bind:key="index"
              class="justify-center align-baseline"
            >
              <v-card outlined flat style="width: 100%">
                <v-row no-gutters>
                  <v-col cols="1 pl-2">
                    <v-checkbox
                      flat
                      dense
                      :false-value="false"
                      :true-value="true"
                      hide-details
                      v-model="augToggles[index]"
                    ></v-checkbox>
                  </v-col>
                  <v-col cols="8" style="width: 100%">
                    <v-text-field
                      style="width: 100%"
                      v-if="augProp != null"
                      :value="getPropertyName(augProp)"
                      filled
                      flat
                      dense
                      disabled
                      disable-lookup
                      single-line
                      hide-details
                      background-color="deep-orange lighten-5"
                      color="grey"
                    >
                    </v-text-field>
                  </v-col>
                  <v-col cols="3" class="">
                    <v-text-field
                      :label="$t('ui.item.value')"
                      background-color="grey lighten-5"
                      color="grey"
                      single-line
                      flat
                      hide-details
                      clearable
                      dense
                      filled
                      v-model="augValues[index]"
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-card>
            </v-row>
          </v-card>
        </v-row>
        <v-row
          no-gutters
          v-if="augCapes != null || augFixed != null || augRoute != null"
          class="pb-2 justify-end pr-2 pt-2"
        >
          <v-btn
            @click="applyAugs"
            small
            color="red lighten-2 white--text"
            elevation="0"
            >{{ $t("ui.item.apply") }}</v-btn
          >
        </v-row>
      </v-col>
    </v-row>
  </v-card>
</template>

<script lang="ts">
import xiUtils from "@/mixins/xiutils";
import { Component, Mixins, Prop, Vue, Watch } from "vue-property-decorator";
import {
  AugProperty,
  AugSet,
  CalculatedAugInfo,
  Equipment,
  EquipSet,
} from "@/@types/equip-set";

@Component
export default class AugmentSelector extends Mixins(xiUtils) {
  name = "AugmentSelector";
  @Prop({ default: null }) readonly equipSetInfo!: EquipSet | null;
  @Prop({ default: "" }) readonly slotName!: string;
  @Prop({ default: false }) readonly dirty!: boolean;

  @Prop() readonly propDict!: Map<string, string[]>;
  @Prop() readonly cateDict!: Map<string, string[]>;

  result: CalculatedAugInfo[] = [];

  augValues: (number | undefined)[] = [];
  augProps: (AugProperty | undefined)[] = [];
  augToggles: boolean[] = [];
  editEquipID = 0;

  @Watch("dirty")
  @Watch("equipSetInfo", { deep: true, immediate: true })
  @Watch("slotName")
  equipChanged(): void {
    //this.augValues.splice(0);
    //this.augProps.splice(0);
    //this.augToggles.splice(0);

    if (this.equipSetInfo == null) {
      return;
    }
    const equipBase = this.equipSetInfo[this.slotName];
    if (equipBase == null) {
      return;
    }
    const equip = equipBase.Equip;
    if (equip.Id == this.editEquipID) {
      return;
    }
    this.editEquipID = equip.Id;
    if (equip.AugCape != null) {
      this.augValues.splice(equip.AugCape.length);
      this.augProps.splice(equip.AugCape.length);
      for (let i = 0; i < equip.AugCape.length; i++) {
        // hacks
        Vue.set(this.augValues, i, undefined);
        Vue.set(this.augProps, i, undefined);
      }
    } else if (equip.AugFixed != null) {
      this.augValues.splice(equip.AugFixed.Augs.length);
      this.augProps.splice(equip.AugFixed.Augs.length);
      this.augToggles.splice(equip.AugFixed.Augs.length);
      for (let i = 0; i < equip.AugFixed.Augs.length; i++) {
        Vue.set(this.augValues, i, equip.AugFixed.Augs[i].Value);
        Vue.set(this.augProps, i, equip.AugFixed.Augs[i]);
        Vue.set(this.augToggles, i, true);
      }
    } else if (equip.AugRoute != null) {
      this.augValues.splice(equip.AugRoute[0].Augs.length);
      this.augProps.splice(equip.AugRoute[0].Augs.length);
      this.augToggles.splice(equip.AugRoute[0].Augs.length);
      for (let i = 0; i < equip.AugRoute[0].Augs.length; i++) {
        Vue.set(this.augValues, i, equip.AugRoute[0].Augs[i].Value);
        Vue.set(this.augProps, i, equip.AugRoute[0].Augs[i]);
        Vue.set(this.augToggles, i, true);
      }
    }
  }

  public augChanged(event: any, index: number): void {
    if (event != null && event.Value != null) {
      Vue.set(this.augValues, index, event.Value);
      Vue.set(this.augProps, index, event);
    } else {
      Vue.set(this.augValues, index, undefined);
      Vue.set(this.augProps, index, undefined);
    }
  }

  public routeChanged(event: any): void {
    this.augValues.splice(0);
    this.augProps.splice(0);
    this.augToggles.splice(0);
    if (event != null && event.Augs != null) {
      for (let i = 0; i < event.Augs.length; i++) {
        this.augValues.push(event.Augs[i].Value);
        this.augProps.push(event.Augs[i]);
        this.augToggles.push(true);
        // hacks
        Vue.set(this.augValues, i, event.Augs[i].Value);
        Vue.set(this.augProps, i, event.Augs[i]);
        Vue.set(this.augToggles, i, true);
      }
    }
  }

  public applyAugs(): void {
    this.result = [];

    for (let i = 0; i < this.augProps.length; i++) {
      const augCape = this.augProps[i];
      if (augCape != null) {
        if (this.augToggles.length > i && !this.augToggles[i]) {
          continue;
        }
        this.addToResult(augCape, this.augValues[i]);
      }
    }
    this.$emit("apply", this.result);
  }

  public addToResult(aug: AugProperty, value: number | undefined): void {
    let isAdded = false;
    for (const prop of aug.PropIDs) {
      for (const p of this.result) {
        if (
          p.CateID == aug.CatID &&
          p.PropID == prop.toString() &&
          p.Unit == aug.Unit &&
          p.MainHand == aug.MainHand
        ) {
          if (p.Value != null && value != null) {
            p.Value += value;
          } else if (p.Value == null && value != null) {
            p.Value = value;
          }
          isAdded = true;
        }
      }
    }
    if (isAdded) {
      return;
    }
    for (const prop of aug.PropIDs) {
      this.result.push({
        CateID: aug.CatID,
        PropID: prop.toString(),
        Unit: aug.Unit,
        Value: value,
        MainHand: aug.MainHand,
      });
    }
  }

  get labelName(): string {
    return this.$t("ui.equipSlot." + this.slotName).toString();
  }

  get hasAug(): boolean {
    return (
      this.source?.AugCape != null ||
      this.source?.AugFixed != null ||
      this.source?.AugRoute != null
    );
  }

  get AugString(): string[] {
    let result: string[] = [];
    if (this.source == null) {
      return result;
    }
    if (this.source.AugCape != undefined) {
      for (const augs of this.source.AugCape) {
        let rsub = "";
        for (const aug of augs.Augs) {
          rsub +=
            this.getAugString(
              this.$i18n.locale,
              aug,
              this.propDict,
              this.cateDict
            )?.replace(" ", "&nbsp;") + " &#8203;";
        }
        result.push(rsub);
      }
    }

    if (this.source.AugRoute != undefined) {
      for (const augs of this.source.AugRoute) {
        let rsub = "";
        for (const aug of augs.Augs) {
          rsub +=
            this.getAugString(
              this.$i18n.locale,
              aug,
              this.propDict,
              this.cateDict
            )?.replace(" ", "&nbsp;") + " &#8203;";
        }
        result.push(rsub);
      }
    }

    if (this.source.AugFixed != undefined) {
      const augs = this.source.AugFixed;
      let rsub = "";
      for (const aug of augs.Augs) {
        rsub +=
          this.getAugString(
            this.$i18n.locale,
            aug,
            this.propDict,
            this.cateDict
          )?.replace(" ", "&nbsp;") + " &#8203;";
      }
      result.push(rsub);
    }

    return result;
  }

  public getPropertyName(aug: AugProperty): string {
    let result = "";
    let loci = this.$i18n.locale == "ja" ? 0 : 1;
    if (aug.MainHand === true) {
      result += this.$t("ui.item.mainhand") + ":";
    }
    if (aug.CatID != null) {
      const cats = this.cateDict.get(aug.CatID.toString());
      if (cats != null) {
        result += cats[loci] + ":";
      }
    }
    for (const propID of aug.PropIDs) {
      const propS = this.propDict.get(propID.toString());
      if (propS != null) {
        result += propS[loci] + " / ";
      }
    }

    result = result.slice(0, -3);

    if (aug.Value != null) {
      if (aug.Value >= 0) {
        result += " (+" + aug.Value + ")";
      } else {
        result += "(" + aug.Value + ")";
      }
    }

    return result;
  }

  get itemName(): string {
    if (this.equipSetInfo == null) {
      return "";
    }
    const equip = this.equipSetInfo[this.slotName];
    if (equip != null) {
      if (this.$i18n.locale == "ja") {
        return equip.Equip.Jp;
      }
      return equip.Equip.En;
    }
    return "";
  }

  get itemTypeName(): string | null {
    if (this.equipSetInfo == null) {
      return null;
    }
    const equip = this.equipSetInfo[this.slotName];
    if (equip == null) {
      return null;
    }
    const locale = this.getItemTypeName(equip.Equip);
    return locale == null ? null : (this.$t(locale) as string);
  }

  get itemDescription(): string | null {
    if (this.$root.$i18n.locale == "ja") {
      return this.source?.JpDescription ?? null;
    } else {
      return this.source?.EnDescription ?? null;
    }
  }

  get isRare(): boolean {
    if (this.equipSetInfo == null) {
      return false;
    }
    const equip = this.equipSetInfo[this.slotName];
    if (equip == null) {
      return false;
    }
    return equip.Equip.Flags == undefined
      ? false
      : this.IsRare(equip.Equip.Flags);
  }

  get isEx(): boolean {
    if (this.equipSetInfo == null) {
      return false;
    }
    const equip = this.equipSetInfo[this.slotName];
    if (equip == null) {
      return false;
    }
    return equip.Equip.Flags !== undefined && this.IsEx(equip.Equip.Flags);
  }

  get jobsString(): string {
    if (this.equipSetInfo == null) {
      return "";
    }
    const equip = this.equipSetInfo[this.slotName];
    if (equip == null) {
      return "";
    }
    let result = "";
    const jobs = this.jobList();
    for (const job of jobs) {
      if ((equip.Equip as any)["Is" + job.abb] == true) {
        result += job.name + " ";
      }
    }
    return result;
  }

  get source(): Equipment | undefined {
    if (this.equipSetInfo == null) {
      return undefined;
    }
    const equip = this.equipSetInfo[this.slotName];
    if (equip == null) {
      return undefined;
    }
    return equip.Equip;
  }

  get iconUrl(): string {
    if (this.equipSetInfo == null) {
      return "";
    }
    const equip = this.equipSetInfo[this.slotName];
    if (equip == null) {
      return "";
    }
    if (equip.Equip == null) {
      return "";
    }
    return this.getItemIconUrl(equip.Equip.Id);
  }

  get augCapes(): AugSet[] | undefined {
    if (this.equipSetInfo == null) {
      return undefined;
    }
    const equip = this.equipSetInfo[this.slotName];
    if (equip == null) {
      return undefined;
    }
    return equip.Equip.AugCape;
  }

  get augRoute(): AugSet[] | undefined {
    if (this.equipSetInfo == null) {
      return undefined;
    }
    const equip = this.equipSetInfo[this.slotName];
    if (equip == null) {
      return undefined;
    }
    return equip.Equip.AugRoute;
  }

  get augFixed(): AugSet | undefined {
    if (this.equipSetInfo == null) {
      return undefined;
    }
    const equip = this.equipSetInfo[this.slotName];
    if (equip == null) {
      return undefined;
    }
    return equip.Equip.AugFixed;
  }

  get calculatedAugsString(): string {
    let string = "";
    for (const aug of this.calculatedAugs) {
      string +=
        this.getCalculatedAugString(
          this.$i18n.locale,
          aug,
          this.propDict,
          this.cateDict
        ) + " ";
    }
    return string.trim();
  }

  get calculatedAugs(): CalculatedAugInfo[] {
    if (this.equipSetInfo == null) {
      return [];
    }
    const slotItem = this.equipSetInfo[this.slotName];
    return slotItem == null || slotItem.Aug == null ? [] : slotItem.Aug;
  }
}
</script>

<style scoped></style>
