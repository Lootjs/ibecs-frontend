<template lang="pug">
    .app
        .title
            span День - {{ day }} |
            nuxt-link(to="/reports") Отчет
        .container
            .sheepfold
                .sheepfold__label 1
                sheep(v-for="(i, n) in sheepfolds.sf1", :key="`sf1_${n}`")
            .sheepfold
                .sheepfold__label 2
                sheep(v-for="(i, n) in sheepfolds.sf2", :key="`sf2_${n}`")
            .sheepfold
                .sheepfold__label 3
                sheep(v-for="(i, n) in sheepfolds.sf3", :key="`sf3_${n}`")
            .sheepfold
                .sheepfold__label 4
                sheep(v-for="(i, n) in sheepfolds.sf4", :key="`sf4_${n}`")
        .newGame
            button(@click="newGame", class="newGame__btn") Новая игра
</template>
<style lang="stylus">
    body
        background #fafafa
        .title
            text-align center
            font-size 26px
            a
                color currentColor
                text-decoration none
                &:hover
                    text-decoration none
        .container
            max-width 80%
            margin 0 auto
            display flex
            flex-flow row
            flex-wrap wrap
            .sheepfold
                position relative
                background-image url('/bg.jpg')
                height 300px
                border 20px solid #ccc
                border-image-source url('/b.png')
                border-image-slice 20
                display flex
                justify-content center
                align-items center
                flex-wrap wrap
                flex-basis calc(49% - 46px)
                margin 10px
                box-shadow 1px 4px 6px 0 #504b4b
                &__label
                    position absolute
                    right 4px
                    top 0
                    color #fff
                    font-size 24px
                    text-shadow 1px 1px 1px #000
        .newGame
            display flex
            justify-content center
            &__btn
                border 1px solid #ccc
                background #fff
                padding 7px
                margin-top 10px
                cursor pointer
</style>
<script>
    import Sheep from '~/components/Sheep.vue'

    const axios = require('axios')
    const config = require('~/config.json')
    export default {
        components: {
            Sheep
        },
        data() {
            return {
                day: 1,
                sheepfolds: {
                    sf1: 0,
                    sf2: 0,
                    sf3: 0,
                    sf4: 0,
                }
            }
        },
        computed: {
            isSlaughterDay() {
                return this.day % 10 === 0
            },
            totalSheeps() {
                if (process.browser) {
                    let total = 0
                    for (const [key, value] of Object.entries(this.sheepfolds)) {
                        total += value
                    }
                    return total
                }
            }
        },
        mounted() {
            this.setState()
            setInterval(this.newDay, config.oneDay)
        },
        methods: {
            newGame () {
              axios.delete(`${config.apiHost}/api/cleanState`)
              this.cleanUpData()
              this.firstDayActions()
            },
            setState () {
                let self = this
                axios.get(`${config.apiHost}/api/getState`).then((res) => {
                    if (res.data.day === 1) {
                        self.firstDayActions()
                    } else {
                        self.day = res.data.day
                        self.sheepfolds.sf1 = res.data.sf1
                        self.sheepfolds.sf2 = res.data.sf2
                        self.sheepfolds.sf3 = res.data.sf3
                        self.sheepfolds.sf4 = res.data.sf4
                    }
                })
            },
            firstDayActions () {
                for (let i = 0; i < 10; i++) {
                    this.addSheepToSheepFold(this.getRandomSheepfold(), 1)
                }
                this.balanceSheepfolds()
            },
            cleanUpData () {
                this.day = 1
                this.sheepfolds.sf1 = 0
                this.sheepfolds.sf2 = 0
                this.sheepfolds.sf3 = 0
                this.sheepfolds.sf4 = 0
            },
            newDay() {
                let dead = 0
                let totalSheeps = this.totalSheeps
                let self = this

                this.sheepfolds[this.getNotAloneSheepFold()] += 1
                if (this.isSlaughterDay) {
                    dead = 1
                    this.removeSheepFromSheepFold(this.getNotAloneSheepFold(), 1)
                    this.balanceSheepfolds()
                }

                axios.post(`${config.apiHost}/api/dailyReport`, {
                    day: self.day,
                    total: totalSheeps,
                    dead: self.isSlaughterDay,
                    alive: (totalSheeps - dead),
                    crowded: self.getFilteredSheepfold('crowded'),
                    least: self.getFilteredSheepfold('least'),
                    sf1: self.sheepfolds.sf1,
                    sf2: self.sheepfolds.sf2,
                    sf3: self.sheepfolds.sf3,
                    sf4: self.sheepfolds.sf4,
                })

                this.day++
            },
            addSheepToSheepFold(sheepfold, count) {
                this.sheepfolds[sheepfold] += count
            },
            removeSheepFromSheepFold(sheepfold, count) {
                this.sheepfolds[sheepfold] -= count
            },
            getRandomSheepfold() {
                return 'sf' + this.getRandomNumber()
            },
            getRandomNumber(min = 1, max = 4) {
                let rand = min + Math.random() * (max + 1 - min);
                rand = Math.floor(rand);
                return rand
            },
            getNotAloneSheepFold() {
                if (process.browser) {
                    let folds = []
                    for (const [key, value] of Object.entries(this.sheepfolds)) {
                        if (value > 1) {
                            folds.push({
                                key: key,
                                value: value
                            })
                        }
                    }
                    let keys = Object.keys(folds)
                    return folds[Object.keys(folds)[Math.floor(Math.random() * Object.keys(folds).length)]].key
                }
            },
            getFilteredSheepfold(filter) {
                if (process.browser) {
                    let tmpFold = {}
                    for (const [key, value] of Object.entries(this.sheepfolds)) {
                        let isFilteredItem = false

                        if (Object.keys(tmpFold).length === 0) {
                            tmpFold = {
                                key: key,
                                value: value
                            }
                        }
                        else if (filter === 'least' && value < tmpFold.value) {
                            isFilteredItem = true
                        }
                        else if (filter === 'crowded' && value > tmpFold.value) {
                            isFilteredItem = true
                        }

                        if (isFilteredItem) {
                            tmpFold = {
                                key: key,
                                value: value
                            }
                        }
                    }
                    return tmpFold.key
                }
            },
            balanceSheepfolds() {
                let least = this.getFilteredSheepfold('least')
                let crowded = this.getFilteredSheepfold('crowded')
                if (this.sheepfolds[least] === 0 || this.sheepfolds[least] === 1) {
                    this.addSheepToSheepFold(least, 1)
                    this.removeSheepFromSheepFold(crowded, 1)
                }
            }
        }
    }
</script>