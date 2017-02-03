<template>
    <div @click="swapSound">
        <div v-show="playing">
            <slot name="on"></slot>
        </div>
        <div v-show="!playing">
            <slot name="off"></slot>
        </div>
    </div>
</template>

/*!
* vue-loader v0.0.1 (https://github.com/johnnyGoo/vue-loader)
* Author: Johnny chen
*
* Copyright 2013-2015 Johnny chen
*/
<script>

    if (!window.Smart) {
        throw 'VueMusicPlayerButton required smart.js'
    }
    var Smart = window.Smart;
    var Sound = Smart.Sound;
    var Event = Smart.Event;
    var WechatShareClass;

    // 注册
    export default {
        // 声明 props
        props: {
            autoPlay: {
                type: Boolean,
                default: false
            }, src: {
                type: String,
                default: ''
            }, loop: {
                type: Boolean,
                default: true
            }, mp3: {type: String, default: ''}
        },
        data: function () {
            return {
                music: null,
                playing: false,
                wechatShare: false,
                autoPlayTouchTime: 0
            }
        },
        watch:{
          playing:function (nv,ov) {
              this.$emit('update-state', this.playing);
          }

        },

        methods: {
            syncState: function () {
                this.playing = this.music.playing();
            },
            swapSound: function () {
                if (this.playing) {
                    if (this.autoPlayTouchTime <= new Date().getTime() - 800) {
                        this.music.pause();
                    }else{
                        this.autoPlayTouchTime=0
                    }
                } else {
                    this.music.play();

                }
                this.syncState();
            },
            fixAutoPlay: function () {
                var me = this;
                if (this.wechatShare) {
                    var share = new WechatShareClass();
                    share.playSound(this.music, function (playing) {
                        me.syncState();
                        if (!playing) {
                            function touchPlay() {
                                me.autoPlayTouchTime = new Date().getTime();
                                me.music.play();
                                me.syncState();
                                rm()
                            }
                            var rm = Event.windowEvent('touchstart', touchPlay);


                        }
                    })
                }else{
                    me.music.play();
                    me.syncState();
                }
            }
        },
        computed: {},
        ready: function () {

            if(typeof(WechatShare)!="undefined" || window.WechatShare){
                WechatShareClass=WechatShare || window.WechatShare;
            }
            this.music = new Sound({path: this.src, loop: this.loop});
            if (WechatShareClass) {
                this.wechatShare = true;
            }
            if (this.autoPlay) {
                this.fixAutoPlay();
            }

        }


    }


</script>