<template>
  <v-card
    class="mx-auto"
    color="#26c6da"
    dark
    max-width="400"
    :loading="loading"
  >
    <v-card-title>
      <img class="udon-image" src="~assets/img/udon.png" alt="" />
      <span class="text-h6 font-weight-light">Marugame Gacha</span>
    </v-card-title>

    <v-card-text class="text-h5 font-weight-bold text-center">
      金額を入力してください
    </v-card-text>

    <div class="text-field">
      <v-text-field
        v-model="inputTotalPrice"
        :rules="rules"
        label="金額(3000円以内)"
        outlined
        clearable
      />
    </div>

    <v-card-actions>
      <v-row align="center" justify="center">
        <v-btn
          class="ma-2"
          :loading="loading"
          :disabled="loading"
          color="secondary"
          @click="getData"
        >
          抽選する
          <template #loader>
            <span class="white-text">抽選中...</span>
          </template>
        </v-btn>
        <v-btn
          class="ma-2"
          :loading="loading"
          :disabled="isDisableNext"
          color="secondary"
          @click="pushResult"
        >
          次の人に回す
        </v-btn>
      </v-row>
    </v-card-actions>

    <div class="result-section">
      <div v-if="result.length === 0">
        <v-card-text class="font-weight-bold text-center">
          {{ "該当のメニューはありません" }}
        </v-card-text>
      </div>

      <div 
        v-else
        class="menu-section"
      >
        <div v-for="(result, index) in result" :key="index">
          <v-btn
              icon
              color="white"
              class="refresh-icon"
              @click="changeMenu(result)"
            >
              <v-icon>mdi-cached</v-icon>
            </v-btn>
          <span class="result-text">
            {{ result.name }} / {{ `${result.price} 円` }}
          </span>
        </div>
        <div class="result-total-price">
          {{ `合計金額 ${resultTotalPrice} 円` }}
        </div>
      </div>
    </div>

    <div 
      class="menu-section"
    >
    <div class="history-section" v-for="(results, historyIndex) in resultHistory" :key="historyIndex">
      <div class="history-text">
        {{ `${historyIndex + 1} 番目の人の結果` }}
      </div>
      <div v-for="(result, index) in results" :key="index">
          <span class="result-history-text">
            {{ result.name }} / {{ `${result.price} 円` }}
          </span>
        </div>
      </div>
    </div>

    <v-card-actions>
      <v-list-item class="grow">
        <v-list-item-avatar color="grey darken-3">
          <v-img
            class="elevation-6"
            alt=""
            src="https://avataaars.io/?avatarStyle=Transparent&topType=ShortHairShortCurly&accessoriesType=Prescription02&hairColor=Black&facialHairType=Blank&clotheType=Hoodie&clotheColor=White&eyeType=Default&eyebrowType=DefaultNatural&mouthType=Default&skinColor=Light"
          ></v-img>
        </v-list-item-avatar>

        <v-list-item-content>
          <v-list-item-title>K-Ishikawa</v-list-item-title>
        </v-list-item-content>
      </v-list-item>
    </v-card-actions>
  </v-card>
</template>

<script>
import shuffle from "lodash/shuffle"

export default {
  data: () => ({
    inputTotalPrice: "",
    resultTotalPrice: 0,
    data: [],
    result: [],
    resultHistory: [],
    loader: null,
    loading: false,
    rules: [
      (v) => !!v || "",
      (v) => /[0-9]/.test(v) || "半角数字で入力してください",
      (v) => (!!v && 3000 >= v) || `3000円以内で入力してください`,
    ],
  }),
  watch: {
    loader() {
      const l = this.loader
      this[l] = !this[l]

      setTimeout(() => (this[l] = false), 300)
      this.loader = null
    },
  },
  computed: {
    isDisableNext() {
      return this.result.length === 0
    }
  },
  methods: {
    async getData() {
      if (this.inputTotalPrice === "") return
      this.loader = "loading"
      if (this.data.length !== 0) {
        // 結果をリセットする
        this.reset()
        this.lottery()
        return
      }
      const ref = this.$fire.database.ref("menu")
      await ref.on(
        "value",
        (snapshot) => {
          this.data = snapshot.val()
          this.data.shift()
          this.lottery()
        },
        (errorObject) => {
          console.log("The read failed: " + errorObject.name)
        }
      )
    },
    lottery(price = null) {
      if(price) {
        var balance = price
      } else {
        var balance = this.inputTotalPrice
      }
      var newArray = []
      var target = []
      var price = 0

      while (true) {
        newArray = this.data.filter((v) => v.price <= balance)
        if (newArray.length === 0) break
        target = shuffle(newArray).slice(0, 1)
        this.result.push(...target)
        price = target[0].price
        balance = balance - price
        this.resultTotalPrice = this.resultTotalPrice + price
      }
    },
    changeMenu(result) {
      // 除外した個数のカウント
      var countArray = this.result.filter((v) => v.name == result.name)
      var exculudedCount = countArray.length

      var execulutedArray = this.result.filter((v) => v.name !== result.name)
      this.result = execulutedArray
      this.resultTotalPrice = this.resultTotalPrice - result.price * exculudedCount
      this.lottery(this.inputTotalPrice - this.resultTotalPrice)
    },
    reset() {
      this.result = []
      this.resultTotalPrice = 0
    },
    pushResult(){
      this.resultHistory.push(this.result)
      this.reset()
    }
  },
}
</script>

<style scoped>
.udon-image {
  width: 32px;
  padding: 0 10px 0 0;
}

.text-field {
  padding: 0 20px;
}

.button-field {
  /* padding-left: 30px; */
  text-align: center;
}

.white-text {
  color: white;
}

.menu-section {
  padding: 5px 0px 5px 60px;
}

.result-text {
  text-align: left;
  font-weight: bold;
  padding-left: 10px;
  width:80%;
}

.result-total-price {
  text-align: right;
  font-weight: bold;
  padding: 5px 10px 10px 0;
}

.result-section {
  padding-top: 15px;
}

.history-section {
  padding-bottom: 10px;
  font-weight: bold;
}

.history-text {
  text-align: left;
  text-decoration:underline
}

.result-history-text {
  font-size: 16px;
}
</style>
