<template>
  <section class="chat">
    <div class="chatdisplay" id="chatdisplay">
      <div class="chatitem" v-for="chat in chats" :key="chat.id">
        <div>{{chat.user}}</div>
        <div>: {{chat.chatinput}}</div>
        <div>{{chat.createdAt}}</div>
      </div>
    </div>
    <div class="chatinput">
      <p v-if="!user"> please login to join chat </p>
      <chatform v-if="user" :schema='schema' :model='model'></chatform>
    </div>
  </section>
</template>

<script>
  import VueForm from 'vue-form-generator';
  import moment from 'moment'
  import jQuery from 'jquery'
  import firebaseApp from '../assets/config'
  const db = firebaseApp.database()
  
  export default {
    name: 'chat',
    props: ['user'],
    components: {
      "chatform": VueForm.component
    },
    created() {
      this.getChats()
    },
    mounted() {
      jQuery(document).ready(function($) {
        $("#chatdisplay").scrollTop(function() {
          return this.scrollHeight;
        });
  
        setTimeout(() => {
          $("#chatdisplay").fadeOut("slow")
        }, 5000);
  
        $('.chat').mouseleave(function() {
          $("#chatdisplay").fadeOut("fast")
        });
  
        $('.chat').mouseenter(function() {
          $("#chatdisplay").fadeIn("slow")
        });
      });
    },
    data() {
      return {
        chats: [],
        model: {
          user: this.user.firstName,
          chatinput: '',
          createdAt: moment(new Date()).format('DD-MMM hh:mm')
        },
        schema: {
          fields: [{
            type: 'input',
            inputType: 'text',
            model: 'chatinput',
            placeholder: 'type chat ...',
            maxlength: 50,
            buttons: [{
                classes: 'btn',
                label: 'Send',
                onclick: () => {
                  this.submitChat()
                }
              },
              {
                classes: 'btn',
                label: '[ ]',
                onclick: function() {
                  jQuery('.chatdisplay').toggle('fast');
                }
              }
            ]
          }]
        }
      }
    },
    methods: {
      getChats() {
        db.ref('/db/globalChat').on('value', snapshot => {
          if (snapshot.val()) {
            this.chats = Object.values(snapshot.val())
          }
        })
      },
      submitChat() {
        try {
          db.ref(`/db/globalChat/`).push({
            user: this.user.firstName,
            chatinput: this.model.chatinput,
            createdAt: moment(new Date()).format('DD-MMM-YY hh:mm')
          })
        } catch (error) {
          console.log(error)
        }
        this.model.chatinput = ''
      }
    }
  }
</script>

<style scoped>
  .chat {
    display: grid;
    position: fixed;
    bottom: 0px;
    right: 10%;
    width: 500px;
    background-color: rgb(255, 255, 255);
    padding: 10px 10px 0 10px;
    max-height: 400px;
    border-radius: 10px 10px 0 0;
    box-shadow: 0 0 5px 2px rgb(202, 202, 202);
  }
  
  .chatdisplay {
    max-height: 300px;
    overflow: hidden;
    margin-bottom: 15px;
    overflow: auto;
  }
  
  .chatitem {
    display: grid;
    grid-template-columns: 1fr 5fr 2fr;
    text-align: left;
  }
  
   ::-webkit-scrollbar {
    display: none;
  }
</style>
