<template>
  <v-app id="inspire">
    <sidebar />
    <v-main>
      <h1>{{ room? room.name : "" }}</h1>
      <v-container
        class="py-8 px-6"
        fluid
      >
        <v-row>
          <v-col
            v-for="card in cards"
            :key="card"
            cols="12"
          >
            <v-card>
              <v-subheader>{{ card }}</v-subheader>

              <v-list two-line>
                <template v-for="(data,index) in messages">
                  <v-list-item

                    :key="index"
                  >
                    <v-list-item-avatar color="grey darken-1">
                      <v-img :src="data.photoURL"></v-img>
                    </v-list-item-avatar>

                    <v-list-item-content>
                      <v-list-item-subtitle class="massage">
                        {{data.message}}
                      </v-list-item-subtitle>
                    </v-list-item-content>
                  </v-list-item>

                  <v-divider
                    v-if="index !== 6"
                    :key="`divider-${index}`"
                    inset
                  ></v-divider>
                </template>
              </v-list>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
      <v-container fluid>
        <v-textarea
          label="send message"
          no-resize
          rows="3"
          v-model="body"
          auto-grow
        ></v-textarea>
      </v-container>
      <v-btn
        class="mr-4"
        @click="submit"
        :disabled="invalid"
      >
        submit
      </v-btn>
      <v-btn @click="clear">
        clear
      </v-btn>
    </v-main>
  </v-app>
</template>

<script>
import firebase from "@/firebase/firebase"
import Sidebar from '@/components/layouts/Sidebar'
export default {
  components: {
    Sidebar
  },
  async created(){
    this.roomId = this.$route.query.room_id;
    console.log("roomId",this.roomId);

    const roomRef = firebase.firestore().collection("rooms").doc(this.roomId);
    const roomDoc = await roomRef.get();
    this.room = roomDoc.data();

    if(!roomDoc.exists){
      await this.$router.push('/');
    }

    // const snapshot = await roomRef.collection('messages').orderBy('createdAt', 'asc').get()

    // snapshot.forEach(doc =>{
    //   console.log(doc.data());
    //   this.messages.push(doc.data())
    // })

  },
  data: () => ({
    messages: [
      // {message: "message 1"},
      // {message: "message 2"},
      // {message: "message 3"},
      // {message: "message 4"},
      // {message: "message 5"},
    ],
    roomId: '',
    room: null,
    cards: ['Today'],
    drawer: null,
    links: [
      ['mdi-inbox-arrow-down', 'Inbox'],
      ['mdi-send', 'Send'],
      ['mdi-delete', 'Trash'],
      ['mdi-alert-octagon', 'Spam'],
    ],
    body: '',
    auth: null,
  }),
  mounted(){
    this.auth = JSON.parse(sessionStorage.getItem('user'))
    console.log(this.auth)

    const roomRef = firebase.firestore().collection('rooms').doc(this.roomId)
    roomRef.collection('messages').orderBy('createdAt', 'asc')
    .onSnapshot(snapshot => {
      snapshot.docChanges().forEach(change => {
        console.log('new message' , change.doc.data())
        this.messages.push(change.doc.data())
      })
    })
  },
  computed:{
    invalid(){
      if(!this.body){
        return true;
      }else{
        return false;
      }
    }
  },
  methods:{
    clear(){
      console.log('clear');
      this.body = "";
    },
    submit(){

      const roomRef = firebase.firestore().collection('rooms').doc(this.roomId)
      console.log(roomRef)
      roomRef.collection('messages').add({
        message: this.body,
        name: this.auth.displayName,
        photoURL: this.auth.photoURL,
        createdAt: firebase.firestore.Timestamp.now(),
      })
      .then(result => {
        console.log('success',result)
        this.body = "";
      })
      .catch(error => {
        console.log('fail',error)
        alert('メッセージの送信に失敗しました')
      })
    },
  }
}
</script>

<style scoped>
.massage{
  text-align: left;
}

</style>>