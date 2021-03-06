<template>
    <div class="send-bar" v-mdl>
        <div class="mdl-progress mdl-js-progress mdl-progress__indeterminate" :style="{ display: loading ? '' : 'none' }" v-mdl></div>
        <div v-if="$store.state.loaded_media" class="preview" v-mdl>
            <div class="overlay">
                <button class="media-clear mdl-button mdl-js-button mdl-button--colored mdl-button--fab mdl-js-ripple-effect" :style="{ background: send_color }" @click="removeMedia">
                    <i class="material-icons">clear</i>
                </button>
            </div>
            <img :src="media_blob" />
        </div>
        <div class="send-bar-inner" id="sendbar">
            <div class="entry mdl-textfield mdl-js-textfield" :class="is_dirty" v-mdl>
                <textarea class="mdl-textfield__input disabled" type="text" id="message-entry" @keydown.shift.enter.stop @keydown.enter.prevent.stop="dispatchSend" v-model="message"></textarea>
                <label class="mdl-textfield__label" for="message-entry">{{ $t('sendbar.type') }}</label>
            </div>
            <!-- fab with correct colors will be inserted here -->
            <button class="send mdl-button mdl-js-button mdl-button--fab mdl-button--mini-fab mdl-button--colored mdl-js-ripple-effect" :style="{ background: send_color }" id="send-button" @click="dispatchSend">
                <i class="material-icons md-18 material-icons-white">send</i>
            </button>
        </div>
    </div>
</template>

<script>
import Vue from 'vue'
import { Api } from '@/utils'

export default {
    name: 'Sendbar',
    props: ['threadId', 'onSend', 'loading'],

    mounted () {
        window.addEventListener('resize', this.updateEmojiMargin)
        this.$wrapper = document.querySelector("#wrapper");
        this.$sendbar = document.querySelector("#message-entry");

        this.$store.state.msgbus.$on('hotkey-emoji', this.toggleEmoji);
    },

    destroy () {

    },

    data () {
        return {
            message: "",
            emojiStyle: {
                position: "absolute",
                left: 0,
                bottom: "70px",
                width: "18em",
            },
            perLine: 6,
            set: 'twitter',
            sheetSize: 32,
            skin: 3,
            show_emoji: false,
            $wrapper: null,
            $sendbar: null,
        }
    },

    methods: {
        /**
         * dispatchSend
         * Handles send and enter press
         * Also handles shift modifier
         * @param e - event object
         */
        dispatchSend(e) { // Dispatch send message when clicked

            // the shift key will be used to toggle between the send and return functionality, based
            // on the users "enter to send" preference, in settings.

            // case one: enter to send=off, no shift key -> return line
            // case two: enter to send=off, shift key -> send message
            // case three: enter to send=on, no shift key -> send message
            // case four: enter to send=on, shift key -> return line

            if (e instanceof KeyboardEvent && (
                  (e.shiftKey && this.$store.state.enter_to_send) ||
                  (!e.shiftKey && !this.$store.state.enter_to_send))) { // return line

                // Get start/end of selection for insert location
                const start = e.target.selectionStart;
                const end =  e.target.selectionEnd;

                // Overwrite selection with newline
                this.message = this.message.substr(0,start)
                    + "\n" + this.message.substr(end, this.message.length)

                // Set new location of selection to start of old selection
                // Wait until next tick to ensure the new message gets rendered
                Vue.nextTick(() =>
                    e.target.setSelectionRange(start + 1, start + 1)
                );

                return; // Full stop
            }

            // If message is empty, we're done
            if (this.message.length <= 0 && !this.$store.state.loaded_media)
                return false;

            // Send message to handler
            this.onSend(this.message);

            // Clear message
            this.message = "";
        }
    },

    computed: {
        send_color () {
            if (this.$store.state.theme_use_global) {
                return this.$store.state.theme_global_accent;
            } else {
                return this.$store.state.colors_accent;
            }
        },
        is_dirty () { // Is dirty fix for mdl
            if (this.message.length > 0)
                return "is-dirty";
            return "";
        },
        media_blob () { // creates url object from media blob
            return window.URL.createObjectURL(this.$store.state.loaded_media)
        }
    },

    watch: {
        '$route' (to) { // Update thread on route change
            this.message = "";
        }
    },
    components: {

    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
@import "../../assets/scss/_vars.scss";

    #emoji {
        background: url("../../assets/images/ic_mood.png") no-repeat;
        background-size: cover;
    }

    #attach {
        background: url("../../assets/images/ic_attach.png") no-repeat;
        background-size: cover;
        margin-top: 18px;
        width: 24px;
        height: 24px;
    }

    body.dark {
      #emoji {
          background: url("../../assets/images/ic_mood_white.png") no-repeat;
          background-size: cover;
      }

      #attach {
          background: url("../../assets/images/ic_attach_white.png") no-repeat;
          background-size: cover;
      }
    }

    .send-bar {
        height: auto;
        width: 100%;
        margin: auto;
        position: fixed;
        bottom: 0%;
        clear: both;
        transition: ease-in-out width $anim-time;

        .mdl-progress {
            width: 100%;
        }
    }

    .mdl-textfield {
        padding: 0px 0;
        padding-top: 20px;
    }

    .mdl-textfield__input {
        min-height: 48px;
        text-align: right;
        transition: height ease-in-out $anim-time;
        outline: none;
    }

    .mdl-textfield__label {
        text-align: right;
    }

    .mdl-textfield__label:after {
        background-color: rgba(0,0,0,0);
        bottom: 1px;
        height: 1px;
    }

    .send-bar-inner {
        background: #fafafa;
        height: 100%;
        margin: auto;
        padding-left: 60px;
        padding-right: 60px;
        padding-top: 4px;
        padding-bottom: 4px;
        box-shadow: -0px -0px 3px rgba(0, 0, 0, .15);

        .entry {
            width: calc(100% - 64px);
            margin-top: -4px;
            font-size: 20px;
        }

        .send {
            float: right;
            margin-top: 8px;
        }

        .attach-button {
            float: left;
            text-align: center;
            margin-top: 15px;
            min-width: 0px;
            width: 28px;
            height: 28px;
            opacity: .56;
            margin-right: 8px;
        }

        .emoji-button {
            float: left;
            text-align: center;
            margin-top: 18px;
            min-width: 0px;
            width: 24px;
            height: 24px;
            opacity: .56;
            margin-right: 16px;
        }
    }

    #message-entry {
        height: 48px;
        resize: none;
    }

    #emoji-wrapper {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        z-index: 10;
    }

    body.dark {
        .preview {
            background: rgb(55,66,72);

            .overlay {
                background: linear-gradient(to bottom, rgba(55,66,72,0) 95%,rgba(55,66,72,1) 100%);
            }
        }

        .send-bar-inner {
            background: #374248;
            color: #fff;
        }

        .mdl-textfield__label {
            color: #fff !important;
        }
    }

    body.black {
        .preview {
            background: rgb(0,0,0);

            .overlay {
                background: linear-gradient(to bottom, rgba(0,0,0,0) 95%,rgba(0,0,0,1) 100%);
            }
        }

        .send-bar-inner {
            background: #000000;
        }
    }

</style>
