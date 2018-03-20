<template>
    <div class="wrap">
        <header id="uiHead" class="ui-head" v-if="showHead">
            <div class="ui-head-in">
                <div class="ui-head-l" @click="back()"><span class="ui-head-btn1"></span></div>
                <div class="ui-head-m"><h2 class="ui-head-tit">{{title}}</h2></div>
                <div class="ui-head-r"><span class="ui-head-btn2"></span></div>
            </div>
        </header>
        <div class="news-detail">
            <div class="zj-list zjwz-info fixed-title" v-bind:class="{'hide':!showfloattitle}">
                <expertlist2 :expertlist="[expertinfo]" :issport="1" :isprofile="1"></expertlist2>
            </div>
            <div class="l-full l-scroll-y" v-on:scroll="showTitle($event)">
                <div id="slidediv">
                    <div class="art-tit">{{detail.title}}</div>
                    <div class="art-time">{{detail.publishtime}}</div>
                </div>
                <div class="zj-list zjwz-info">
                    <expertlist2 :expertlist="[expertinfo]" :issport="1" :isprofile="1"></expertlist2>
                </div>
                <div class="article-body" v-bind:class="{hide:!detail.freecontent}">
                    <div v-if="detail.projectcontent.proinfo">
                        <div class="zjtj-cont" v-if="detail.type==1" v-for="v in detail.projectcontent.proinfo">
                            <div class="zjtj-tit">{{v.ls}}&nbsp; &nbsp;{{v.stime.slice(5,16)}}&nbsp; &nbsp;{{TypeList[1][v.type]}}</div>
                            <div class="zjtj-detail">
                                <div class="team-info">
                                    <p><sub>主</sub>{{v.home}}</p>
                                    <p>{{v.score?v.score:'VS'}}</p>
                                    <p>{{v.away}}</p>
                                </div>
                                <div class="game-info">
                                    <p class="game-info-l green" v-if="v.type==2">{{v.rangqiu}}</p>
                                    <div class="game-info-r" v-html="getZqPei(v)"></div>
                                </div>
                            </div>
                        </div>
                        <div class="zjtj-cont" v-if="detail.type==3" v-for="v in detail.projectcontent.proinfo">
                            <div class="zjtj-tit">{{v.ls}}&nbsp; &nbsp;{{v.stime.slice(5,16)}}&nbsp; &nbsp;{{TypeList[3][v.type]}}</div>
                            <div class="zjtj-detail">
                                <div class="team-info">
                                    <p>{{v.away}}</p>
                                    <p>{{v.score?v.score.replace(/(\d+):(\d+)/,'$2,$1'):'VS'}}</p>
                                    <p>{{v.home}}</p>
                                </div>
                                <div class="game-info">
                                    <div class="game-info-r" v-html="getLqPei(v)"></div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="zjtj-cont zjtj-zucai"
                         v-if="detail.type==2&&detail.projectcontent.project&&detail.projectcontent.matchinfo">
                        <div class="zjtj-tit">
                            <ul>
                                <li>足彩胜负彩{{detail.projectcontent.project.lottype==2?'(任九)':''}}第<span class="red">{{detail.projectcontent.project.expect}}</span>期
                                </li>
                                <li>方案金额：<span class="red">{{detail.projectcontent.project.money}}</span>元</li>
                            </ul>
                        </div>
                        <div class="zucai_scroll">
                            <table cellpadding="0" cellspacing="0" class="zucai_show">
                                <tbody>
                                <tr>
                                    <th>场次</th>
                                    <th v-for="item in [1,2,3,4,5,6,7,8,9,10,11,12,13,14]">{{item}}</th>
                                </tr>
                                <tr>
                                    <th>主队</th>
                                    <td v-for="item in detail.projectcontent.matchinfo.list"><span class="zucai_team">{{item.homesx}}</span>
                                    </td>
                                </tr>
                                <tr v-html="getZc()">
                                </tr>
                                </tbody>
                            </table>
                        </div>
                        <div class="zucai_info">
                            <span class="fl">开奖时间：{{detail.projectcontent.matchinfo.resulttime.slice(5,10)}}</span>
                            <span class="fr">截止时间：{{detail.projectcontent.matchinfo.encashtime.slice(5,16)}}</span>
                        </div>
                    </div>
                    <div v-if="(detail.type==4||detail.type==5)&&detail.projectcontent.code">
                        <div class="zjtj-wrap" v-bind:class="{'zjtj-end':detail.projectcontent.opencode}">
                            <div class="zjtj-title">{{detail.type==4?'双色球':'大乐透'}}{{detail.projectcontent.expect}}期推荐
                            </div>
                            <div class="zjtj-cont">
                                <div class="zjtj-tit">
                                    <ul>
                                        <li>注数：<i class="red">{{detail.projectcontent.zhushu}}</i></li>
                                        <li>方案金额：<i class="red">{{detail.projectcontent.money}}</i></li>
                                        <li class="red" v-if="detail.projectcontent.prizes>0">
                                            已中{{detail.projectcontent.prizes}}等奖
                                        </li>
                                    </ul>
                                </div>
                                <div class="zjtj-detail"
                                     v-html="getBallHtml(detail.projectcontent.code,detail.projectcontent.opencode)"></div>
                            </div>
                        </div>
                    </div>
                    <div class="art-txt" v-html="detail.freecontent"></div>
                    <div class="art-txt" v-html="detail.paycontent"></div>
                    <div class="pay-article-box" v-if="detail.paytype==1&&ispayed==0&&detail.canpublic==0">
                        <div class="how-much"><em>{{detail.slogan}}</em></div>
                        <p class="msg" v-if="endstrtotime<0" v-html="endline + '，请根据需要购买'"></p>
                        <p class="msg" v-if="endstrtotime>0&&endstrtotime!=10801" v-html="endline"></p>
                        <div class="pay-article">
                            <div class="pay-tit-wrap">
                                <div class="pay-tit" v-if="!overtime" v-on:click="pay(1,detail.paymoney);">
                                    <div class="pay-tit-detail">{{(parseFloat(detail.paymoney)).toFixed(2)}}元 解锁<span
                                            v-if="detail.gametag">推荐玩法：{{detail.gametag}}</span></div>
                                    <em class="suo-icon"></em>
                                </div>
                                <div class="pay-no-btn" v-if="overtime">文章已过期</div>
                            </div>
                            <p class="sqfw"><span class="mianz-a"></span>购买即代表您已经阅读并同意<em v-on:click="agreement=true">《付费阅读协议》</em>
                            </p>
                        </div>
                    </div>
                    <div class="pay-article-box  reward-box"
                         v-bind:class="{hide:detail.paytype==1&&ispayed==0&&detail.canpublic==0}">
                        <p class="msg" v-if="endstrtotime!=10801" v-html="endline"></p>
                        <div class="btn-reward" v-on:click="showTip=true"><span>打赏</span></div>
                        <div class="num-reward" v-if="paycount>0||tipcount>0">已有{{paycount>0?(paycount +'人付费 '):''}}
                            {{tipcount>0?(tipcount +'人打赏 '):''}}啦！<span v-if="userpayed>0">您已打赏<i class="red">{{userpayed}}</i>元</span>
                        </div>
                        <div class="num-reward" v-if="detail.canpublic==1&&ispayed==0">赛后公开文章不显示用户购买信息</div>
                        <div class="pay-user-icon" v-if="detail.canpublic==0">
                            <ul>
                                <li v-for="v in payuserlist"><img v-bind:src="v.icon" alt="打赏用户头像"> <em
                                        v-if="v.paytype==0"></em></li>
                            </ul>
                        </div>
                    </div>
                    <p class="bbzq">观点建议仅供参考<span v-if="detail.ispublic==1">,专家选择赛后/开奖后公开文章</span></p>
                </div>
            </div>

            <div v-bind:class="{'hide':!showTip}">
                <div class="alert-loading" v-on:click="showTip=false;"></div>
                <div class="alert-das">
                    <div class="close-alert close-das" v-on:click="showTip=false"></div>
                    <div class="title">打赏{{expertinfo.nickname}}</div>
                    <p class="das-txt">{{ranTipListText[tipmoney]}}</p>
                    <div class="das-money">￥{{tipmoney}}<span class="random-money" v-on:click="getTipMoney">随机</span>
                    </div>
                    <div class="btn-das" v-on:click="pay(0,tipmoney);showTip=false">打赏</div>
                </div>
            </div>


            <div v-bind:class="{'hide':!showPaySuccess}">
                <div class="alert-loading" v-on:click="showPaySuccess=false"></div>
                <div class="das-suc-alert fufei-suc">
                    <div class="msg">解锁成功！</div>
                    <div class="das-suc-img"><img src="//www.500cache.com/mobile/touch/images/gzpt/fufei_suc.png"
                                                  alt="打赏成功"></div>
                    <div class="info">
                        <p class="gray">千万别错过了投注时间哦~</p>
                        <p v-if="detail.expiretime>0" v-html="endline"></p>
                    </div>
                    <div class="btn-ok" v-on:click="showPaySuccess=false;">我知道了</div>
                </div>
            </div>

            <div v-bind:class="{'hide':!showTipSuccess}">
                <div class="alert-loading" v-on:click="showTipSuccess=false;"></div>
                <div class="das-suc-alert">
                    <div class="msg">感谢您的欣赏，祝您红单连连！</div>
                    <div class="das-suc-img"><img src="//www.500cache.com/mobile/touch/images/gzpt/das_suc.png"></div>
                    <div class="author" v-if="!isconcerned">
                        <p class="gray">这么欣赏他，不如关注一下</p>
                        <div class="zj-list zjwz-info">
                            <expertlist2 :expertlist="[expertinfo]" :issport="0" :isprofile="0"></expertlist2>
                        </div>
                    </div>
                    <div class="btn-ok" v-on:click="showTipSuccess=false;">好的</div>
                </div>
            </div>
            <div v-bind:class="{hide:!agreement}">
                <div class="ui-alert-bg" v-on:click="agreement=false"></div>
                <div class="ffxy-alert">
                    <div class="close-alert" v-on:click="agreement=false"></div>
                    <div class="ffxy-tit">《500彩票付费阅读协议》</div>
                    <div class="ffxy-detail">
                        <h2>一、风险提示</h2>
                        <p>1、本付费阅读内容为作者个人提供，文中观点结论和建议仅供参考，“500彩票网”不对投注建议的确定性做出判断或审核，亦不为此承担任何法律责任。</p>
                        <p>2、付费阅读内容仅作为一种文章的欣赏或者意见的参考，不应作为彩票投注决策的唯一参考因素，任何交易风险或不可预测的其他风险用户应当自行承担。</p>
                        <p>3、本付费阅读内容没有考虑到个别用户特殊的投注目标、财务状况或需求，而用户应对自身情况做出基本判断。</p>
                        <p>
                            4、付费阅读中可能带有煽动性、确定性的推荐并不构成任何承诺，我们不建议作者给出承诺，而用户亦应该在使用本付费阅读功能时，应注意甄别、慎重、正确使用本内容，独立进行投注决策，防止被误导。 </p>
                        <p>5、本付费阅读内容仅供购买人本人使用，任何机构和个人均不得以任何形式复制、引用、转载。一经发现，本公司将保留随时追究其法律责任的权利。</p>
                        <h2>二、协议修改</h2>
                        <p>
                            1、根据互联网的发展和有关法律、法规及规范性文件的变化，或者因业务发展的需要，500彩票网有权对本协议的条款作出修改或变更，一旦本协议内容发生变动，500彩票网将会直接在网站上公布修改之后的协议内容，该公布行为视为500彩票网已经通知用户修改内容。 </p>
                        <p>2、如果不同意500彩票网对本协议相关条款所做的修改，用户有权并应当停止使用，如果用户继续使用，则视为用户同意接受500彩票网对本协议相关条款所做的修改。 </p>
                        <h2>三、免责声明</h2>
                        <p>
                            “500彩票网”仅基于独立、客观、公正和审慎的原则提供本资讯分享平台，信息均由用户或第三方提供，本公司对这些信息的准确性和完整性不作任何保证。在任何情况下，本资讯中的信息或所表述的意见并不构成对任何人的投注建议，本公司及其雇员不对使用本资讯及其内容所引发的任何直接或间接损失负任何责任。</p>
                        <p>
                            “500彩票网”提供本资讯分享平台，不意味着对于作者或用户发布的内容应当负有完整全面的审查义务，但若任意第三方有证据证明本资讯分享平台上作者的内容有任何侵犯第三方合法权益的行为，“500彩票网”可以采取合理且必要的手段保护第三方的权益不受侵犯。</p>
                        <p>本规则最终解释权归500彩票网所有。</p>
                        <div class="btn-reward btn-sure" v-on:click="agreement=false;">确认阅读</div>
                    </div>
                </div>

            </div>

            <div class="get-more" v-if="noreferer">
                <a v-on:click="jumpUrl('/openplatform')">更多推荐</a>
            </div>

        </div>
        <lighttip v-bind:content="tip.content" v-bind:show="tip.show" @close="tip.tipclose"></lighttip>
    </div>
</template>

<script>
    import CommView from '~pages/openplatform/CommView.vue'
    import expertlist2 from '~components/openplatform/expertlist2.vue'
    import lighttip from '~components/openplatform/lighttip.vue'
    import platform from '~common/platform.js'
    import payutil from '~common/payutil.js'
    export default {
        name: "detail",
        data() {
            return {
                title: '文章详情',
                aid: 0,
                ranTipListText: {
                    '0.88': '赏你8毛8',
                    '1.88': '赏你188，我要发发发',
                    '2.88': '288，爱发发',
                    '5.20': '给你520，跟你永不停',
                    '6.66': '赏你666，奖金biubiubiu',
                    '8.88': '赏你888，一路发发发',
                    '9.99': '赏你9块9，跟你到永久',
                },
                TypeList: {1: {1: '竞彩', 2: '让球', 3: '亚盘', 4: '大小'}, 3: {2: '让分', 4: '大小分'}},
                ranTipList: [0.88, 1.88, 2.88, 0.88, 6.66, 1.88, 9.99, 0.88, 2.88, 0.88, 1.88, 2.88, 5.20, 0.88, 0.88, 8.88, 1.88, 1.88, 1.88, 2.88],
                tipmoney: 0.88,
                agreement: false,
                showTip: false,
                showPay: false,
                showTipSuccess: false,
                showPaySuccess: false,
                showfloattitle: false,
                endstrtotime: 10801,
                noreferer: false,
                endlinetimer: null,
                overtime: false,
                endline: '',
            }
        },
        asyncData({store, route,cookies}) {
            let aid = route.params.aid
            return store.dispatch('getDetailData', {aid,cookies})
        },
        mixins: [CommView],
        mounted() {
            this.aid = parseInt(this.$route.params.aid)
            platform.ready(() => {
                this.noreferer = document.referrer == '' && (!window.EsApp)
                let script = document.createElement('script')
                script.src = '//m.500.com/js/??esun/pay/esPay.js'
                document.body.appendChild(script)
            })

            this.shareDetail.title = "大神" + this.expertinfo.nickname + "推单"
            this.shareDetail.content = this.detail.title
            this.setRead(this.detail.aid)

            let expiretime = this.detail.expiretime
            if (expiretime > 0 && this.endlinetimer == null) {
                let t = new Date(expiretime * 1000)
                if (['4', '5'].indexOf(this.detail.type) > -1) { //处理投注时间
                    t.setHours(19)
                    t.setMinutes(30)
                    expiretime = (t - 0) / 1000
                }
                //设置不能购买
                if ((['1', '3'].indexOf(this.detail.type) == -1 && t - this.timesys < 0) || (['1', '3'].indexOf(this.detail.type) > -1 && t.setHours(t.getHours() + 1) - this.timesys < 0)) {
                    this.overtime = true;
                }
                let text = '距{type}时间:<span {red}>{word}</span>'
                let type = this.detail.type
                text = text.replace('{type}', ['1', '3'].indexOf(type) > -1 ? '比赛' : '截止')
                this.endlinetimer = setInterval(() => {
                    this.time = parseInt(this.time) + 1
                    this.endstrtotime = expiretime - this.time
                    if (this.endstrtotime < 10800) {
                        text = text.replace('{red}', 'class="red"')
                    }
                    if (this.endstrtotime < 0) {
                        this.endline = text.replace('{word}', '已开始/结束')
                        clearInterval(this.endlinetimer)
                        return
                    }
                    let limit = this.endstrtotime, s = ('0' + limit % 60).slice(-2), m = '00', h = '00'
                    limit = limit / 60 >> 0
                    if (limit > 0) {
                        m = limit % 60
                        m = ('0' + m).slice(-2)
                        h = limit / 60 >> 0
                        h = ('0' + h).slice(-2)
                    }
                    this.endline = text.replace('{word}', h + ':' + m + ':' + s)
                }, 1000)
            }
            if (this.$route.params.hasOwnProperty('orderid')) {
                let orderid = this.$route.params.orderid
                if (this.$route.params.inapp > 0) {
                    let inapp = this.$route.params.inapp
                    location.href = 'rabbit' + (inapp == 1 ? '' : 'zx') + '://'
                }
                let ispaydata = payutil.ispay(this.$route.params.orderid)
                if (ispaydata) {
                    this.afterPay(ispaydata)
                }
            }
            this.loadimg()
            this.record()
        },
        components: {
            expertlist2,lighttip
        },
        computed: {
            expertinfo() {
                return this.$store.state.detail.expertinfo
            },
            detail() {
                return this.$store.state.detail.detail
            },
            isself() {
                return this.$store.state.detail.isself
            },
            ispayed() {
                return this.$store.state.detail.ispayed
            },
            tipcount() {
                return this.$store.state.detail.tipcount
            },
            paycount() {
                return this.$store.state.detail.paycount
            },
            userpayed() {
                return this.$store.state.detail.userpayed
            },
            payuserlist() {
                return this.$store.state.detail.payuserlist
            },
            isconcerned() {
                return this.$store.state.detail.isconcerned
            },
        },
        methods: {
            afterPay(data) {
                if (data.ordertype == 1) {
                    this.showPaySuccess = true;
                } else {
                    this.showTipSuccess = true;
                }
                this.$store.dispatch('getDetailData',{aid:this.aid})
            },
            loadimg() {
                var containerlist = document.querySelectorAll('.art-txt')
                containerlist.forEach(container => {
                    let imglist = container.querySelectorAll('[data-src]')
                    imglist.forEach(img => {
                        if (img.getAttribute("data-opz") != "1") {
                            img.setAttribute("data-opz", "1")
                        }
                        let src = img.getAttribute("data-src")
                        src = src.split('?')[0]
                        let endfix = src.split(".").pop()
                        if (endfix) {
                            img.src = src + "?!414." + (this.webp ? "webp" : endfix)
                            img.onload = function () {
                                img.width = img.width * window.devicePixelRatio
                            }
                        }
                    })
                })
            },
            record() {
                this.$store.dispatch('setRecord', this.aid)
            },
            getLqPei(v) {
                let [html, selected, selected2] = ['', 'class="selected"', 'class="selected notin"']
                let re = /^(\d)@([\d.]+)#([\d.]+)$/
                let select = v.hasOwnProperty('result') && v.result != 1 && v.result != 3 ? selected2 : selected
                try {
                    switch (parseInt(v.type)) {
                        case 2: {
                            let m = v.tinfo.match(re)
                            let r = m[1]
                            let pankou = v.pankou
                            pankou = pankou > 0 ? ('+' + pankou) : pankou
                            html = '<p ' + (r == 0 ? select : '') + '>主输<span>' + m[3] + '</span></p><em>' + pankou + '</em><p ' + (r == 3 ? select : '') + '>主赢<span>' + m[2] + '</span></p>'
                            break
                        }
                        case 4: {
                            let m = v.tinfo.match(re)
                            if (m.length == 4) {
                                let r = m[1]
                                html = '<p ' + (r == 0 ? select : '') + '>小球<span>' + m[3] + '</span></p><em>' + v.pankou + '</em><p ' + (r == 3 ? select : '') + '>大球<span>' + m[2] + '</span></p>'
                            }
                            break
                        }
                    }
                } catch (e) {
                }
                return html
            },
            getZqPei(v) {
                let [html, selected, selected2] = ['', 'class="selected"', 'class="selected notin"']
                let re = /^(\d)@([\d.]+)#([\d.]+)$/
                let select = v.hasOwnProperty('result') && v.result != 1 && v.result != 3 ? selected2 : selected
                try {
                    switch (parseInt(v.type)) {
                        case 1:
                        case 2:
                            if (v.pei) {
                                let pei = v.pei.split(',')
                                let choice = v.choice.split(',')
                                html = '<p ' + (choice.indexOf('3') > -1 ? select : '') + ' >主胜<span>' + pei[0] + '</span></p><p ' + (choice.indexOf('1') > -1 ? select : '') + '>平<span>' + pei[1] + '</span></p><p ' + (choice.indexOf('0') > -1 ? select : '') + '>主负<span>' + pei[2] + '</span></p>';
                            }
                            break
                        case 3: {
                            let m = v.tinfo.match(re)
                            let r = m[1]
                            let pankou = v.pankou
                            pankou = pankou > 0 ? ('+' + pankou) : pankou
                            html = '<p ' + (r == 3 ? select : '') + '>主赢<span>' + m[2] + '</span></p><em>' + pankou + '</em><p ' + (r == 0 ? select : '') + '>主输<span>' + m[3] + '</span></p>'
                            break
                        }
                        case 4: {
                            let m = v.tinfo.match(re)
                            if (m.length == 4) {
                                let r = m[1]
                                html = '<p ' + (r == 3 ? select : '') + '>大球<span>' + m[2] + '</span></p><em>' + v.pankou + '</em><p ' + (r == 0 ? select : '') + '>小球<span>' + m[3] + '</span></p>'
                            }
                            break;
                        }
                    }
                } catch (e) {
                }
                return html;
            },
            pay(type, money) {
                var inapp = window.EsApp ? (EsApp._protocal && EsApp._protocal.indexOf('zx') > -1 ? 2 : 1) : 0
                var h = location.href,
                    backurl = location.origin + '/openplatform/detail/' + this.aid + '/{orderid}/' + inapp
                payutil.pay({
                    data: {
                        did: this.aid,
                        ordermoney: money,
                        ordertype: type,
                        backurl: backurl
                    },
                    success: this.afterPay,
                    fail(data){
                        this.tip.show =true
                        this.tip.content = data.msg
                    }
                });
            },
            getConcern (event) {
                var _this = this;
                var act = 'follow';
                if (this.isconcern) act = 'unfollow';
                Comm.ajax(act, {
                    data: {uname: _this.expertinfo.uname},
                    success: function (data) {
                        if (data.code == '100') {
                            _this.isconcern = _this.isconcern == 1 ? 0 : 1;
                        } else if (data.code == '-101') {
                            Comm.login();
                        }
                    }
                });
                event.stopPropagation();
            },
            getTipMoney () {
                let l = this.ranTipList.length,
                    m = this.ranTipList[Math.floor(Math.random() * l)].toFixed(2)
                while (m == this.tipmoney) {
                    m = this.ranTipList[Math.floor(Math.random() * l)].toFixed(2)
                }
                this.tipmoney = m;
            },
            showTitle (event) {
                if (event.target.scrollTop >= document.querySelector('#slidediv').offsetHeight) {
                    this.showfloattitle = true;
                } else {
                    this.showfloattitle = false;
                }
            },
            getBallHtml (code, opencode) {
                code = code.split('|')
                if (code.length == 2) {
                    let [red, blue, ophtml] = [code[0].split(','), code[1].split(','), '']
                    if (opencode) {
                        let opcode = opencode.split('|'),
                            opred = opcode[0].split(','),
                            opblue = opcode[1].split(',');

                        ophtml = `<tr><td colspan="2">开奖号码：<span class="nums"><span class="red"><i>${opred.join('</i><i>')}</i></span><span class="blue"></span>${opblue.join('</i><i>')}</span></td></tr>`
                        redhtml = `${red.map(value => {
                            `<i ${opred.indexOf(value) > -1 ? 'class="ball-slt"' : ''}>${value}</i>`
                        }).join('')}`
                        redhtml = `${blue.map(value => {
                            `<i ${opblue.indexOf(value) > -1 ? 'class="ball-slt"' : ''}>${value}</i>`
                        }).join('')}`
                    } else {
                        redhtml = `<i>${red.join('</i><i>')}</i>`;
                        bluehtml = `<i>${blue.join('</i><i>')}</i>`;
                    }
                    return `<table cellpadding="0" cellspacing="0" width="100%" class="ball-show">
                        <tr><td width="40" valign="top">${this.detail.type == 4 ? '红球' : '前区'}</td><td><span class="ball ball_red">${redhtml}</span> </td></tr>
                        <tr class="ball-show-after"><td>${this.detail.type == 4 ? '蓝球' : '后区'}</td><td><span class="ball ball_blue">${bluehtml}</span></td></tr>${ophtml}</table>`;
                }
            },
            getZc () {
                let [html, isopen, opencode] = ['<th>推荐</th>', false, []]
                if (this.detail.projectcontent.project.opencode) {
                    isopen = true
                    opencode = this.detail.projectcontent.project.opencode.split(',')
                }
                this.detail.projectcontent.project.code.split(',').forEach((v, k) => {
                    v = v.split('')
                    if (isopen) {
                        let i = v.indexOf(opencode[k])
                        i > -1 ? (v[i] = '<em class="red">' + v[i] + '</em>') : ''
                    }
                    v = v.join(',')
                    html += '<td class="b">' + v + '</td>'
                })
                return html
            },
            /*  setShare(){
                  this.shareDetail =  location.href
                  if(platform.isWeixin){
                      share(this.shareDetail)
                  }else if(platform.isApp){
                      EsApp.invoke("sharebtn", {isShow: this.showShare[this.type]?1:0});
                      EsApp.on("sharebtnclicked", function () {
                          EsApp.send("share", {common:this.shareDetail});
                      }.bind(this));
                  }else{
                      import nativeShare from  '~common/nativeshare.js'
                      nativeShare.setShareData({
                          title: this.shareDetail.title,
                          desc: this.shareDetail.content,
                          link:this.shareDetail.url,
                          icon:this.shareDetail.img,
                      })
                  }
              },
              share:function(){ //浏览器情况
                  try {
                      nativeShare.call()
                  } catch (err) {
                      this.commshare = {
                          show:true,
                          callback:function(){
                              this.commshare.show = false;
                          }.bind(this)
                      }
                  }
              }*/
        },
    }
</script>
