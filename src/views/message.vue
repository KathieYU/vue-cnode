<template>
    <div id="message">
        <na-head title='消息'></na-head>
        <div class="message-tab">
            <span :class="{active:!tabFlag}" @click='tabFlag = 0'>未读消息</span>
            <span :class="{active:tabFlag}" @click='tabFlag = 1'>已读消息</span>
        </div>
        <div class="message-container" v-if='message.has_read_messages' ref='messageContainer'>
            <div v-if='!tabFlag' class="mark-container">
                <div class="mark-all" v-if='message.hasnot_read_messages.length'>
                    <span @click='markAll'>标记为已读</span>
                </div>
                <ul v-if='message.hasnot_read_messages.length'>
                    <li v-for='(message,index) in message.hasnot_read_messages' class="item" :key="index">
                        <div class="item-content">
                            <router-link :to="{name:'detail',params:{id:message.topic.id}}">
                                <h1>{{message.topic.title}}</h1>
                            </router-link>
                            <div class="item-content-bottom">
                                <div class="bottom-avatar">
                                    <router-link :to="`/user/${message.author.loginname}`">
                                        <img :src="message.author.avatar_url" alt="" width="45px" height='45px'>
                                    </router-link>
                                </div>
                                <div class="bottom-info">
                                    <p class="tips" v-if="message.type === 'at'">{{message.author.loginname}}评论中@了你:</p>
                                    <p v-else class="tips">{{message.author.loginname}}参与了主题的评论:</p>
                                    <p class="markdown-body info-content" v-html='message.reply.content'></p>
                                </div>
                            </div>
                        </div>
                    </li>
                </ul>
                <div v-else class="no-data">
                    <i class="fa fa-file-text-o fa-2x"></i>
                    <span>暂无数据</span>
                </div>
            </div>
            <div v-if='tabFlag'>
                <ul v-if='message.has_read_messages.length'>
                    <li v-for='(message,index) in message.has_read_messages' class="item" :key="index">        
                        <div class="item-content">
                            <router-link :to="{name:'detail',params:{id:message.topic.id}}">
                                <h1>{{message.topic.title}}</h1>
                            </router-link>
                            <div class="item-content-bottom">
                                <div class="bottom-avatar">
                                    <router-link :to="`/user/${message.author.loginname}`">
                                        <img :src="message.author.avatar_url" alt="" width="45px" height='45px'>
                                    </router-link>
                                </div>
                                <div class="bottom-info">
                                    <p class="tips" v-if="message.type === 'at'">{{message.author.loginname}}评论中@了你:</p>
                                    <p v-else class="tips">{{message.author.loginname}}参与了主题的评论:</p>
                                    <p class="markdown-body info-content" v-html='message.reply.content'></p>
                                </div>
                            </div>
                        </div>
                    </li>
                </ul>
                <div v-else class="no-data">
                    <i class="fa fa-file-text-o fa-2x"></i>
                    <span>暂无数据</span>
                </div>
            </div>
        </div>
        <na-menu></na-menu>
    </div>
</template>

<script>
    import naHead from '../components/header'
    import naMenu from '../components/menu'
    import naBackToTop from '../components/backtotop'

    export default {
        data () {
            return {
                accesstoken: '',
                message: {},
                tabFlag: 0
            }
        },
        mounted () {
            this.accesstoken = this.$route.params.accesstoken
            let url = 'https://cnodejs.org/api/v1/messages'
            this.$http.get(url, {params: {accesstoken: this.accesstoken}}).then(res => {
                if (res.body.success) {
                    this.message = res.body.data
                } else {
                    this.$alert('获取数据失败', 1000)
                }
            })
        },
        methods: {
            markAll () {
                let confirm = window.confirm('是否标记所有未读消息为已读')
                if (!confirm) return
                let url = 'https://cnodejs.org/api/v1/message/mark_all'
                this.$http.post(url, {accesstoken: this.accesstoken}).then(res => {
                    if (res.body.success) {
                        let length = this.message.hasnot_read_messages.length
                        this.$alert('标记成功', 1000)
                        for (let i = 0; i < length; i++) {
                            this.message.has_read_messages.push(this.message.hasnot_read_messages[i])
                        }
                        this.message.hasnot_read_messages = []
                    }
                })
            }
        },
        components: {
            naHead,
            naMenu,
            naBackToTop
        }
    }
</script>

<style lang='stylus'>
    tabHeight = 45px
    .message-tab
        display flex
        height tabHeight
        padding-top 46px
        line-height tabHeight
        span
            flex-basis 50%
            border 1px solid #f5f5f5
            text-align center
            font-size 14px
            &.active
                border-bottom 2px solid #1dd388
    .message-container
        overflow-y auto
        height calc(100vh - 93px)
        .mark-container
            padding-top 40px
        .mark-all
            position fixed
            top (tabHeight * 2 + 1)
            left 0
            right 0
            border 1px solid #f5f5f5
            line-height 40px
            text-align right
            background #fff
            font-size 12px    
            span
                display inline-block
                height 100%
                padding-right 20px
        .item
            padding 10px
            border-bottom 1px solid #f5f5f5
            .item-content
                .item-content-bottom
                    display flex
                    .bottom-avatar
                        flex-basis 45px
                        margin 14px 10px 0 0
                    .bottom-info
                        display flex
                        flex 1
                        flex-wrap wrap
                        max-width 84%
                        p.tips
                            flex-basis 100%
                            padding 10px 0px
                            color #999 
                        p.info-content
                            flex-basis 100%
                            overflow-y auto
                        a
                            display inline-block
                h1
                    overflow hidden
                    border 1px solid #ddd
                    border-radius 5px
                    padding 5px 20px
                    text-align center
                    white-space nowrap
                    text-overflow ellipsis
            a
                color #333
                display block

        .no-data
            position absolute
            top 30%
            left 50%
            text-align center
            transform translate(-50%, 0)
            span
                display block
                margin-top 10px
</style>