<template>
    <div>
        <Menu />
        <b-row class="date-row">
            <b-col>
                <b-datepicker v-model="listDate" size="sm" class="date-div" offset="-35px" />
            </b-col>
        </b-row>
        <b-card no-body>
            <b-tabs pills card vertical>
                <b-tab title="Playout" active @click="getLog('ffplayout')">
                    <b-container class="log-container">
                        <!-- eslint-disable-next-line -->
                        <pre v-if="currentLog" :inner-html.prop="currentLog | formatStr" class="log-content" />
                    </b-container>
                </b-tab>
                <b-tab title="Decoder" @click="getLog('decoder')">
                    <b-container class="log-container">
                        <!-- eslint-disable-next-line -->
                        <pre v-if="currentLog" :inner-html.prop="currentLog | formatStr" class="log-content" />
                    </b-container>
                </b-tab>
                <b-tab title="Encoder" @click="getLog('encoder')">
                    <b-container class="log-container">
                        <!-- eslint-disable-next-line -->
                        <pre v-if="currentLog" :inner-html.prop="currentLog | formatStr" class="log-content" />
                    </b-container>
                </b-tab>
                <b-tab title="System" @click="getSystemLog()">
                    <b-container class="log-container">
                        <!-- eslint-disable-next-line -->
                        <pre v-if="currentLog" :inner-html.prop="currentLog | formatStr" class="log-content" />
                    </b-container>
                </b-tab>
            </b-tabs>
        </b-card>
    </div>
</template>

<script>
// import { mapState } from 'vuex'
import Menu from '@/components/Menu.vue'

export default {
    name: 'Logging',
    middleware: 'auth',

    components: {
        Menu
    },

    filters: {
        formatStr (text) {
            return text
                .replace(/("\[.*")/g, '<span class="log-cmd">$1</span>')
                .replace(/("\/.*")/g, '<span class="log-path">$1</span>')
                .replace(/(\/[\w\d.\-/]+\n)/g, '<span class="log-path">$1</span>')
                .replace(/((tcp|https?):\/\/[\w\d.:]+)/g, '<span class="log-url">$1</span>')
                .replace(/(\[\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}[0-9,.]+\])/g, '<span class="log-time">$1</span>')
                .replace(/\[INFO\]/g, '<span class="log-info">[INFO]</span>')
                .replace(/\[WARNING\]/g, '<span class="log-warning">[WARNING]</span>')
                .replace(/\[ERROR\]/g, '<span class="log-error">[ERROR]</span>')
                .replace(/\[DEBUG\]/g, '<span class="log-debug">[DEBUG]</span>')
        }
    },

    data () {
        return {
            logName: 'ffplayout',
            currentLog: null,
            listDate: this.$dayjs().format('YYYY-MM-DD')
        }
    },

    computed: {
    },

    watch: {
        listDate (date) {
            this.getLog(this.logName)
        }
    },

    async created () {
        await this.getLog('ffplayout')
    },

    methods: {
        async getLog (type) {
            this.logName = type
            const response = await this.$axios.get(`api/player/log/?type=${type}&date=${this.listDate}`)

            if (response.data.log) {
                this.currentLog = response.data.log
            } else {
                this.currentLog = ''
            }
        },
        async getSystemLog () {
            const response = await this.$axios.post('api/player/system/', { run: 'log' })

            if (response.data.data) {
                this.currentLog = response.data.data
            } else {
                this.currentLog = ''
            }
        }
    }
}
</script>

<style>
.col-auto {
    width: 122px;
}

.tab-content {
    max-width: calc(100% - 122px);
}

.log-container {
    background: #1d2024;
    max-width: 95%;
    padding: 1em;
}

.log-content {
    color: #ececec;
}

.log-time {
    color: #a7a7a7;
}

.log-info {
    color: #51d1de;
}

.log-warning {
    color: #e4a428;
}

.log-error {
    color: #e42e28;
}

.log-debug {
    color: #23e493;
}

.log-path {
    color: #e366cf;
}

.log-url {
    color: #e3d666;
}

.log-cmd {
    color: #f1aa77;
}
</style>
