<template>
  <div>
    <v-list-item :value="source" class="align-center" two-line :ripple="false">
      <v-list-item-content>
        <v-list-item-title class="align-center">
          <v-row>
            <v-col cols="1">
              <v-btn
                class="pa-0 ma-0"
                color="blue lighten-5"
                dark
                elevation="1"
                fab
                width="22px"
                height="22px"
                x-small
                v-on:click="onSelectItem"
                ><v-icon color="blue darken-1">mdi-plus</v-icon>
              </v-btn>
            </v-col>
            <v-col cols="2">
              <v-img
                aspect-ratio="1"
                contain
                :alt="itemName"
                :src="iconUrl"
                max-height="32px"
              ></v-img>
            </v-col>
            <v-col cols="9" class="pl-0">
              <v-row class="align-center pb-1" no-gutters
                >{{ itemName }}
                <v-chip
                  v-show="itemTypeName"
                  class="ml-2 mr-1"
                  color="black"
                  label
                  text-color="white"
                  x-small
                >
                  {{ itemTypeName }}
                </v-chip>
              </v-row>
              <v-row class="align-center pb-1" no-gutters>
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
            </v-col>
          </v-row>
        </v-list-item-title>

        <v-list-item-subtitle class="text-wrap">
          <v-row
            no-gutters
            style="font-size: 7px"
            class="blue-grey--text lighten-5--text pb-1 pt-1"
            >{{ jobsString }}</v-row
          >
          <v-row no-gutters
            ><text-highlight
              :queries="highlightArray"
              :wholeWordMatch="true"
              class="grey--text text--darken-1"
              style="
                word-break: keep-all;
                overflow-wrap: break-word;
                line-height: 1.4em;
              "
              >{{ itemDescription }}
            </text-highlight></v-row
          >
          <v-row v-if="hasAug" no-gutters>
            <v-divider class="mt-1 mb-1"></v-divider>
            <v-expansion-panels accordion multiple tile class="ma-2">
              <v-expansion-panel
                v-for="(item, idx) in AugString"
                :key="idx"
                class="mb-0 pb-0"
              >
                <v-expansion-panel-header
                  class="grey--text text--darken-2"
                  color="grey lighten-4"
                  style="min-height: 15px; height: 15px"
                  >{{ item.label }}</v-expansion-panel-header
                >
                <v-expansion-panel-content
                  class="pt-4 pb-0 mb-0 grey--text text--darken-1"
                  style="word-break: keep-all; overflow-wrap: break-word"
                >
                  <text-highlight
                    :queries="highlightArray"
                    :wholeWordMatch="true"
                    style="
                      word-break: keep-all;
                      overflow-wrap: normal;
                      line-height: 1.4em;
                    "
                    v-html="item.aug"
                  >
                  </text-highlight>
                </v-expansion-panel-content>
              </v-expansion-panel>
            </v-expansion-panels>
          </v-row>
        </v-list-item-subtitle>
        <v-list-item-subtitle>
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
        </v-list-item-subtitle>
      </v-list-item-content>
    </v-list-item>
    <v-divider />
  </div>
</template>

<script lang="ts">
import { Component, Mixins, Prop } from "vue-property-decorator";
import scrollerUtils from "@/mixins/scrollerUtils";
import { Equipment, Limiter } from "@/@types/equip-set";
import xiUtils from "@/mixins/xiutils";

@Component
export default class EquipmentListItem extends Mixins(scrollerUtils, xiUtils) {
  @Prop() readonly index!: number;
  @Prop() readonly source!: Equipment;
  @Prop() readonly limiters!: Limiter[];
  @Prop() readonly propDict!: Map<string, string[]>;
  @Prop() readonly cateDict!: Map<string, string[]>;

  get isRare(): boolean {
    return this.source.Flags == undefined
      ? false
      : this.IsRare(this.source.Flags);
  }

  get iconUrl(): string {
    return this.getItemIconUrl(this.source.Id);
  }

  get isEx(): boolean {
    return this.source.Flags !== undefined && this.IsEx(this.source.Flags);
  }

  get itemTypeName(): string | null {
    const locale = this.getItemTypeName(this.source);
    return locale == null ? null : (this.$t(locale) as string);
  }

  get itemName(): string {
    if (this.$root.$i18n.locale == "ja") {
      return this.source.JpFull;
    } else {
      return this.source.En;
    }
  }

  get jobsString(): string {
    let result = "";
    const jobs = this.jobList();
    for (const job of jobs) {
      if ((this.source as any)["Is" + job.abb] == true) {
        result += job.name + " ";
      }
    }
    return result;
  }

  get itemDescription(): string | null {
    if (this.$root.$i18n.locale == "ja") {
      return this.source.JpDescription ?? null;
    } else {
      return this.source.EnDescription ?? null;
    }
  }

  get highlightArray(): (string | RegExp | null)[] {
    let result = [];
    const lang = this.$i18n.locale;
    for (let limiter of this.limiters) {
      if (limiter.propertyID != null && limiter.isActive) {
        if (limiter.isProp) {
          //console.log(this.propDict);
          const propData = this.propDict.get(limiter.propertyID.toString());
          if (propData == undefined) {
            continue;
          }
          let propName = propData[lang == "ja" ? 0 : 1];
          if (propName == undefined) {
            continue;
          }
          let propNameMatches = propName.match(/[:：](.+)/);
          if (propNameMatches != undefined && propNameMatches.length > 1) {
            propName = propNameMatches[1];
          }
          result.push(
            new RegExp("(^|[ \n\b:：])" + propName + "(?=[\n +\\-\b0-9])", "i")
          );
        }
        if (limiter.isText) {
          let propName = limiter.text;
          result.push(propName);
        }
      }
    }
    return result;
  }

  get hasAug(): boolean {
    return (
      this.source.AugCape != null ||
      this.source.AugFixed != null ||
      this.source.AugRoute != null
    );
  }

  get AugString(): { label: string; aug: string }[] {
    let result: { label: string; aug: string }[] = [];
    if (this.source.AugCape != undefined) {
      for (const augs of this.source.AugCape) {
        let rSub = "";
        for (const aug of augs.Augs) {
          rSub +=
            this.getAugString(
              this.$i18n.locale,
              aug,
              this.propDict,
              this.cateDict
            )?.replace(" ", "&nbsp;") + " &#8203;";
        }
        result.push({
          label: this.$t("ui.aug." + augs.Label).toString(),
          aug: rSub,
        });
      }
    }

    if (this.source.AugRoute != undefined) {
      for (const augs of this.source.AugRoute) {
        let rSub = "";
        for (const aug of augs.Augs) {
          rSub +=
            this.getAugString(
              this.$i18n.locale,
              aug,
              this.propDict,
              this.cateDict
            )?.replace(" ", "&nbsp;") + " &#8203;";
        }
        result.push({
          label: this.$t("ui.aug." + augs.Label).toString(),
          aug: rSub,
        });
      }
    }

    if (this.source.AugFixed != undefined) {
      const augs = this.source.AugFixed;
      let rSub = "";
      for (const aug of augs.Augs) {
        rSub +=
          this.getAugString(
            this.$i18n.locale,
            aug,
            this.propDict,
            this.cateDict
          )?.replace(" ", "&nbsp;") + " &#8203;";
      }
      result.push({
        label: this.$t("ui.aug." + augs.Label).toString(),
        aug: rSub,
      });
    }

    return result;
  }

  public onSelectItem(): void {
    this.dispatch("EquipmentList", "select", this.source);
  }
}
</script>
