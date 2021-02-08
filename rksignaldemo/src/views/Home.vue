<template>
  <div class="home">

    <h2>
      User
    </h2>
    <div>
      <input
        type="text"
        v-model="user"
        placeholder="user name"
      />
    </div>
    <div>
      <input
        type="text"
        v-model="newMessage"
        id="message-box"
        class="form-control"
        placeholder="Type message here..."
        autocomplete="off"
      />
      <button @click="sendMessage">Send message</button>
    </div>
    <h2>Messages</h2>
    <div
      v-for="(message, index) in messages"
      :key="index"
    >
      <strong>{{message.sender}}</strong> {{message.text}}
    </div>
  </div>
</template>

<script>
  import axios from 'axios';

  const signalR = require('@aspnet/signalr');
  const apiBaseUrl = 'https://rkrkfunctionapp20210205151439.azurewebsites.net';

  export default {
    name: 'Home',
    components: {
    },
    props: {
    },
    data: () => ({
      connection: null,
      user: 'change me',
      messages: [],
      newMessage: ''
    }),
    computed: {
    },
    watch: {
    },
    created () {
      this.initializeSignalR();
    },
    methods: {
      initializeSignalR () {
        let api = `${apiBaseUrl}/api`;
        this.connection = new signalR.HubConnectionBuilder()
          .withUrl(api)
          .configureLogging(signalR.LogLevel.Information)
          .build();

        console.log('connecting...', api);
        this.connection.start()
          .then((response) => {
            console.log('connection established', response);
          })
          .catch(logError => {
            console.log(logError);
          });

        this.connection.on('newMessage', this.onNewMessage);
      },
      sendMessage () {
        this.createMessage(this.user, this.newMessage);
      },
      onNewMessage (message) {
        this.data.messages = [...this.data.messages, { ...message }];
      },
      logError (err) {
        console.error('Error establishing connection', err);
      },
      createMessage (sender, messageText) {
        // axios.defaults.headers.common['Access-Control-Allow-Origin'] = '*';
        // axios.defaults.headers.common['Access-Control-Allow-Methods'] = '*';
        // axios.defaults.headers.common['Access-Control-Allow-Credentials'] = 'true';

        return axios.post(`${apiBaseUrl}/api/messages`, {
          // headers: {
          //   // remove headers
          //   'Access-Control-Allow-Origin': '*',
          //   'Access-Control-Allow-Methods': '*',
          //   'Access-Control-Allow-Credentials': 'true',
          //   'Referer': 'http://localhost:8085/'
          // },
          sender: sender,
          text: messageText
        }).then(resp => {
          console.log('message sent', resp);
        });
      }
    }
  };
</script>
