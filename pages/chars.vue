<template>
  <div class="char-panel">
    <div class="detail-list">
      <v-container>
        <v-row>
          <v-col
            ><v-menu offset-y>
              <template #activator="{ on, attrs }">
                <v-btn color="primary" dark v-bind="attrs" v-on="on">
                  {{ chars[currentCharId].char_name }}
                </v-btn>
              </template>
              <v-list>
                <v-list-item v-for="(char, index) in chars" :key="index" link>
                  <v-list-item-title @click="currentCharId = char.id">{{
                    char.char_name
                  }}</v-list-item-title>
                </v-list-item>
              </v-list>
            </v-menu></v-col
          >
          <v-col
            ><v-tabs v-model="currentDetailTabId" right>
              <v-tab> Stat </v-tab>
              <v-tab> Feats </v-tab>
              <v-tab> Powers </v-tab>
              <v-tab> Items </v-tab>
            </v-tabs></v-col
          >
        </v-row></v-container
      >

      <v-tabs-items v-model="currentDetailTabId">
        <v-tab-item v-for="tabOption in tabOptions" :key="tabOption.tab">
          <div v-if="tabOption.tab == 'Stat'">
            <div v-for="stat in chars[currentCharId].charStat" :key="stat.name">
              {{ stat.name }}: {{ stat.value }}
            </div>
          </div>
          <div v-if="tabOption.tab == 'Feats'">
            <div
              v-for="feat in chars[currentCharId].char_feats"
              :key="feat.name"
            >
              <v-btn small block :href="feat.url" target="result">{{
                feat.name
              }}</v-btn>
            </div>
          </div>
          <div v-if="tabOption.tab == 'Powers'">
            <div
              v-for="power in chars[currentCharId].charPowers"
              :key="power.name"
            >
              <v-btn small block :href="power.url" target="result"
                ><v-badge inline left :content="power.usage">
                  {{ power.name }}</v-badge
                ></v-btn
              >
            </div>
          </div>
          <div v-if="tabOption.tab == 'Items'">
            <h4>Equipped</h4>
            <v-divider></v-divider>
            <div
              v-for="(item, index) in chars[currentCharId].charItems"
              :key="'E' + index"
            >
              <div v-if="item.equip == 1">
                <v-btn small block :href="item.url" target="result">{{
                  item.name
                }}</v-btn>
              </div>
            </div>
            <v-divider></v-divider>
            <h4>Consumables</h4>
            <v-divider></v-divider>
            <div
              v-for="(item, index) in chars[currentCharId].charItems"
              :key="'C' + index"
            >
              <div v-if="item.count > 1">
                <v-btn small block :href="item.url" target="result">
                  <v-badge inline :content="item.count"
                    >{{ item.name }}
                  </v-badge></v-btn
                >
              </div>
            </div>
            <v-divider></v-divider>
            <h4>Others</h4>
            <v-divider></v-divider>
            <div
              v-for="(item, index) in chars[currentCharId].charItems"
              :key="'O' + index"
            >
              <div v-if="item.count == 1 && item.equip == 0">
                <v-btn small block :href="item.url" target="result">
                  {{ item.name }}</v-btn
                >
              </div>
            </div>
          </div>
        </v-tab-item>
      </v-tabs-items>
    </div>
    <div class="resultview">
      <iframe id="resultiframe" name="result"></iframe>
    </div>
  </div>
</template>

<script>
import xml2js from 'xml2js'
import axios from 'axios'

export default {
  name: 'CharPanel',
  data() {
    return {
      title: 'Chars',
      currentCharId: 0,
      chars: [
        {
          id: 0,
          char_url: 'https://nonjosh.com/downloads/char/nonjoshiii.dnd4e',
          char_name: '',
          charStat: [],
          char_feats: [],
          charPowers: [],
          charItems: [],
        },
        {
          id: 1,
          char_url: 'https://nonjosh.com/downloads/char/jonahbreather.dnd4e',
          char_name: '',
          charStat: [],
          char_feats: [],
          charPowers: [],
          charItems: [],
        },
        {
          id: 2,
          char_url: 'https://nonjosh.com/downloads/char/michaelcrusader.dnd4e',
          char_name: '',
          charStat: [],
          char_feats: [],
          charPowers: [],
          charItems: [],
        },
        {
          id: 3,
          char_url: 'https://nonjosh.com/downloads/char/ken_sharti (1).dnd4e',
          char_name: '',
          charStat: [],
          char_feats: [],
          charPowers: [],
          charItems: [],
        },
      ],
      currentDetailTabId: null,
      tabOptions: [
        { tab: 'Stat', content: 'Stat Content' },
        { tab: 'Feats', content: 'Feats Content' },
        { tab: 'Powers', content: 'Powers Content' },
        { tab: 'Items', content: 'Items Content' },
      ],
    }
  },
  head() {
    return {
      title: this.title,
    }
  },
  computed: {
    currentChar() {
      return this.chars[this.currentCharId]
    },
  },
  created() {
    this.chars.forEach((char) => {
      axios
        .get(char.char_url)
        .then((response) => {
          const parser = new xml2js.Parser()

          let charJson
          parser.parseString(response.data, (err, rst) => {
            if (err) {
              console.error(err.stack)
            }
            charJson = rst
          })

          // render char name
          char.char_name =
            charJson.D20Character.CharacterSheet[0].Details[0].name[0]

          // render char stat
          const charStat =
            charJson.D20Character.CharacterSheet[0].StatBlock[0].Stat
          charStat.forEach((statObj) => {
            const statObjName = statObj.alias[0].$.name
            const statObjValue = statObj.$.value
            if (
              statObjName.endsWith('Defense') ||
              statObjName.endsWith('AC') ||
              statObjName.endsWith('Initiative') ||
              statObjName.endsWith('Hit Points')
            ) {
              const stat = {}
              stat.name = statObjName
              stat.value = statObjValue
              char.charStat.push(stat)
            }
          })

          // feat list\
          const RulesElementTally =
            charJson.D20Character.CharacterSheet[0].RulesElementTally[0]
              .RulesElement
          RulesElementTally.forEach((RulesElement) => {
            const elementType = RulesElement.$.type
            if (elementType === 'Feat') {
              const feat = {}
              feat.name = RulesElement.$.name
              feat.url = feat.name.replace(
                / \(Strength\)| \(Constitution\)| \(Dexterity\)| \(Intelligence\)| \(Wisdom\)| \(Charisma\)/,
                ''
              )
              feat.url = feat.url.replaceAll(' ', '-').replaceAll("'", '')
              feat.url =
                'https://data.dnd.nonjosh.com/compendium/feat/' +
                feat.url +
                '.html'
              char.char_feats.push(feat)
            }
          })

          // render power list
          const charPowers =
            charJson.D20Character.CharacterSheet[0].PowerStats[0].Power
          charPowers.forEach((powerObj) => {
            const power = {}
            power.name = powerObj.$.name
            power.usage = powerObj.specific[0]._
            power.action = powerObj.specific[1]._
            switch (power.name) {
              case 'Melee Basic Attack':
                power.url =
                  'https://data.dnd.nonjosh.com/compendium/glossary/Basic-Attack.html'
                break
              case 'Ranged Basic Attack':
                power.url =
                  'https://data.dnd.nonjosh.com/compendium/glossary/Basic-Attack.html'
                break
              case 'Bull Rush Attack':
                power.url =
                  'https://data.dnd.nonjosh.com/compendium/glossary/Bull-Rush.html'
                break
              case 'Grab Attack':
                power.url =
                  'https://data.dnd.nonjosh.com/compendium/glossary/Grab.html'
                break
              case 'Opportunity Attack':
                power.url =
                  'https://data.dnd.nonjosh.com/compendium/glossary/Opportunity-Attack.html'
                break
              case 'Second Wind':
                power.url =
                  'https://data.dnd.nonjosh.com/compendium/glossary/Second-wind.html'
                break
              case 'Refire the Forge Attack':
                power.url =
                  'https://data.dnd.nonjosh.com/compendium/power/Refire-the-Forge.html'
                break
              default:
                power.url =
                  'https://data.dnd.nonjosh.com/compendium/power/' +
                  power.name.replaceAll(' ', '-').replaceAll("'", '') +
                  '.html'
            }
            char.charPowers.push(power)
          })
          // sort item list
          char.charPowers.sort(this.sortPower)

          // render item list
          const LootTally =
            charJson.D20Character.CharacterSheet[0].LootTally[0].loot
          LootTally.forEach((lootObj) => {
            // console.log(lootObj);
            const item = {}
            const RulesElement =
              lootObj.RulesElement[lootObj.RulesElement.length - 1]
            item.name = RulesElement.$.name

            const advGear = [
              "Adventurer's Kit",
              "Climber's Kit",
              "Thieves' Tools",
              'Tent',
              'Fire Kit',
              'Torch',
              'Common Meal',
              'Alchemical Reagents (Arcana)',
            ]
            if (advGear.includes(item.name)) {
              return
            }

            item.count = parseInt(lootObj.$.count)

            item.equip = lootObj.$['equip-count'] === '1'
            let url = item.name.replace(
              / \(heroic tier\)| \(paragon tier\)| \(epic tier\)/,
              ''
            )
            if (url.lastIndexOf(' (level') > -1) {
              url = url.substring(0, url.lastIndexOf(' (level'))
            }
            url = url.replace(/ \+1| \+2| \+3| \+4| \+5| \+6|/g, '')
            url = url.replaceAll(' ', '-').replaceAll("'", '')
            url = url.replaceAll(/\(|\)/g, '')
            item.url =
              'https://data.dnd.nonjosh.com/compendium/item/' + url + '.html'

            char.charItems.push(item)
          })
          // sort item list
          char.charItems.sort(this.sortByObjName)
        })
        .catch((err) => {
          // Manage the state of the application if the request
          // has failed
          console.error(err)
        })
    })
  },
  methods: {
    sortPower(a, b) {
      if (a.name < b.name) {
        return -1
      }
      if (a.name > b.name) {
        return 1
      }
      return 0
    },
    sortByObjName(a, b) {
      if (a.name < b.name) {
        return -1
      }
      if (a.name > b.name) {
        return 1
      }
      return 0
    },
  },
}
</script>

<style lang="scss">
.detail-list {
  width: 35vw;
  height: 95vh;
  float: left;
  overflow-y: scroll;
}
.resultview {
  width: 40vw;
  height: 95vh;
  float: left;
}
#resultiframe {
  width: 32vw;
  min-width: 615px;
  height: 100%;
}
.v-btn,
.v-tab {
  text-transform: none;
}
</style>
