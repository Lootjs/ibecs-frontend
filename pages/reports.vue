<template lang="pug">
    .app
        .filter
            h4 Фильтр по дням
            span Укажите с какого дня:
            input(type="number", v-model="moreThan", placeholder="От", min="1")
            span До какого дня:
            input(type="number", v-model="lessThan", placeholder="До")
        table
            thead
                tr
                    th День
                    th общее количество овечек
                    th количество убитых овечек
                    th количество живых овечек
                    th номер самого населённого загона
                    th номер самого менее населённого загона
            tbody
                tr(v-for="report in filteredReports")
                    td {{ report.day }}
                    td {{ report.total }}
                    td {{ report.dead }}
                    td {{ report.alive }}
                    td {{ report.crowded }}
                    td {{ report.least }}
</template>
<script>
    const axios = require('axios')
    const config = require('~/config.json')

    export default {
        data() {
            return {
                moreThan: 1,
                lessThan: 0,
                reports: []
            }
        },
        computed: {
            filteredReports() {
                return this.reports.filter(item => {
                    if (this.lessThan) {
                        let lessThan = this.lessThan < this.moreThan ? 0 : this.lessThan
                        return item.day >= this.moreThan && item.day <= lessThan
                    } else {
                        return item.day >= this.moreThan
                    }
                })
            }
        },
        mounted() {
            let self = this
            let data = axios.get(`${config.apiHost}/api/getReports`).then((res) => {
                self.reports = res.data
                console.log(res.data)
            })
        }
    }
</script>
<style lang="stylus">
    table
        width 100%
        margin 50px 0
        border 1px solid #ccc
        border-spacing 0
        thead
            th
                padding 10px
                font-weight bold
        tbody
            border-top 1px solid #ccc
            tr
                &:hover
                    background-color #e8e8e8
                td
                    padding 10px
                    text-align center
</style>