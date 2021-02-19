<template>
  <v-row justify="center" align="start">
    <v-col cols="2">
      <v-text-field label="Connect Code" v-model="connect_id">
      </v-text-field>
      <v-btn block @click="connect_to_peer">Remote</v-btn>
      <v-divider></v-divider>
      <v-text-field label="My Connection Code" readonly style="margin-top:25vh;"  v-model="my_peer_id"></v-text-field>
      <v-text-field label="Username"  v-model="my_username"></v-text-field>
      <v-btn block @click="create_peer">Connect</v-btn>
      <v-switch
      v-model="alert"
      style="margin-top:100px;"
      label="ALERT Sound "
      ></v-switch>
    </v-col>
    <v-col cols="10">
      <v-tabs
      v-model="tab" >
      <v-tab>
        Chat
      </v-tab>
      <v-tab>
        Video
      </v-tab>
    </v-tabs>
    <v-tabs-items v-model="tab">
      <v-tab-item>
          <chat-window
          :height="screenHeight"
          :theme="theme"
          :styles="styles"
          :current-user-id="currentUserId"
          :room-id="roomId"
          :rooms="rooms"
          :loading-rooms="loadingRooms"
          :messages="messages"
          :messages-loaded="true"
          :show-audio="false"
          :accepted-files="'image/png, image/jpeg'"
          		@open-file="openFile"
          :rooms-loaded="roomsLoaded"
          :room-actions="roomActions"
          :menu-actions="menuActions"
          :room-message="roomMessage"
          @send-message="send_message"
          :single-room="true"
        >
        </chat-window>
      </v-tab-item>
      <v-tab-item>

      </v-tab-item>
    </v-tabs-items>
    </v-col>
  </v-row>
</template>

<script>
  import ChatWindow from 'vue-advanced-chat';
  //import 'vue-advanced-chat/dist/vue-advanced-chat.css';
  import Peer from 'peerjs';

export default {
  components: {
    ChatWindow
  },
  data(){
    return{
      tab: null,
      roomsPerPage: 15,
			rooms: [
                    {
                    roomId: 1,
                    roomName: 'General Room',
                    avatar: '',
                    unreadCount: 1,
                    index: 1,
                    /*
                    lastMessage: {
                      content: 'Last message received',
                      senderId: 1234,
                      username: 'John Doe',
                      timestamp: '10:20',
                      saved: true,
                      distributed: false,
                      seen: false,
                      new: true
                    },
                    */
                   
                    users: [
                     
                    ],
                    typingUsers: []
                    
                  }
      ],
			roomId: 1,
			startRooms: null,
			endRooms: null,
			roomsLoaded: true,
			loadingRooms: false,
			allUsers: [
        /*
                      {
                        _id: '6R0MijpK6M4AIrwaaCY2',
                        username: 'Luke',
                        avatar: ''
                      },
                      {
                        _id: 'SGmFnBZB4xxMv9V4CVlW',
                        username: 'Leia',
                        avatar: ''
                      },
                      {
                        _id: '6jMsIXUrBHBj7o2cRlau',
                        username: 'Yoda',
                        avatar:
                          ''
                      }
                      */
      ],
      currentUserId: 0,
			loadingLastMessageByRoom: 0,
			roomsLoadedCount: false,
			selectedRoom: null,
			messagesPerPage: 100,
			messages: [],
			messagesLoaded: true,
			roomMessage: '',
			startMessages: null,
			endMessages: null,
			roomsListeners: [],
			listeners: [],
			typingMessageCache: '',
			disableForm: false,
			addNewRoom: null,
			addRoomUsername: '',
			inviteRoomId: null,
			invitedUsername: '',
			removeRoomId: null,
			removeUserId: '',
			removeUsers: [],
			roomActions: [],
			menuActions: [],
			styles: { container: { borderRadius: '0px' } },
      peer_local:null,
      peer_connection:null,

      my_peer_id:null,
      connect_id:null,
      my_username:"",
      message_id:0,

      alert:false
    }
  },
  computed: {
		screenHeight() {
			return this.isDevice ? window.innerHeight + 'px' : 'calc(95vh - 80px)'
		}
	},
  methods: {
    send_message({ content, roomId, file, replyMessage }){
      if(this.my_peer_id){
      
      if(this.peer_connection){
        if(file){
          var self =this;
            console.log(content)
             console.log(file)
             var reader = new FileReader();
              reader.readAsDataURL(file.blob); 
              reader.onloadend = function() {
                  var base64data = reader.result;                
                  console.log(base64data);
                  var temp_file = {
                    name: file.name,
                    size:file.size,
                    type: file.type,
                    audio: false,
                    duration: 14.4,
                    url: base64data
                    };
                  self.create_file_message(content,self.my_peer_id,self.my_username, temp_file )
                  self.peer_connection.send({
                    type:"image",
                    name:self.my_username,
                    uid:self.my_peer_id,
                    msg:content,
                    data:temp_file
                  });
              }
             

        }else{
          this.create_message(content,this.my_peer_id,this.my_username);
          this.peer_connection.send({
            type:"text",
            name:this.my_username,
            uid:this.my_peer_id,
            msg:content
          });

        }
      }
      }
    },
    openFile({ message }) {
      console.log(message.file)
			window.open(message.file.url, '_blank')
		},
    connect_to_peer(){
      if(this.connect_id){
        var self = this;
        this.peer_connection = this.peer_local.connect(this.connect_id);
        this.peer_connection.on('open', function() {
          // Receive messages
          self.peer_connection.on('data', function(data) {
            console.log('Received', data);
             if(self.alert){
              var src = './sounds/notif.mp3';
                    var audio = new Audio(src);
                    audio.play();
              }

            if(data.type == "text"){
            
             
                      var user = {
                        _id: data.uid,
                        username: data.name,
                        status: {
                          state: 'online',
                          lastChanged: new Date()
                        }
                      };

                      self.rooms[0].users.push(user) // = [...self.rooms[0].users, user]
                      self.message_id++;
                      self.create_message(data.name+"::"+data.msg,data.uid,data.name)
            }
            if(data.type == "image"){
            self.create_file_message(data.msg,data.uid,self.name,data.data )
          }
          });

          // Send messages
          //self.peer_connection.send('Hello!');
        });
        
      }else{
        alert("Create connection first");
      }
    },
    create_message(msg,id,username){
      var text_msg = {
          _id: this.message_id,
          content: msg,
          senderId: id,
          username: username,
          date: new Date(),
          timestamp: new Date(),
          system: false,
          saved: true,
          distributed: true,
          seen: true,
          disableActions: true,
          disableReactions: true,
          /*
          file: {
            name: 'My File',
            size: 67351,
            type: 'png',
            audio: true,
            duration: 14.4,
            url: ''
          },
          reactions: {
            wink: [
              1234, // USER_ID
              4321
            ],
            laughing: [
              1234
            ]
          }*/
        }
        this.messages.push(text_msg)
    },
    create_file_message(msg,id,username,file){
      var text_msg = {
          _id: this.message_id,
          content: msg,
          senderId: id,
          username: username,
          date: new Date(),
          timestamp: new Date(),
          system: false,
          saved: true,
          distributed: true,
          seen: true,
          disableActions: true,
          disableReactions: true,
          file: file,/*

          {
            name: 'My File',
            size: 67351,
            type: 'png',
            audio: true,
            duration: 14.4,
            url: ''
          }
          reactions: {
            wink: [
              1234, // USER_ID
              4321
            ],
            laughing: [
              1234
            ]
          }*/
        }
        this.messages.push(text_msg)
    },
    create_peer(){
      if(!this.my_username){
        alert("Please add username !");
        return;
      }
      this.get_recieve();
    },
    get_recieve(){

      this.peer_local = new Peer();
      var self = this;
      this.peer_local.on('open', function(id) {
        self.my_peer_id = id;
        //console.log(self.peer_local);
       
        var user = {
                        _id: self.my_peer_id,
                        username: self.my_username,
                        status: {
                          state: 'online',
                          lastChanged: new Date()
                        }
                      };

                      //self.rooms[0].users = [...self.rooms[0].users, user]
                      self.rooms[0].users.push(user)
        self.currentUserId = self.my_peer_id
      });

      this.peer_local.on('connection', (conn) => {
        self.peer_connection  = conn;
        self.peer_connection.on('data', (data) => {
          // Will print 'hi!'
          //console.log(data);
          if(self.alert){
              var src = './sounds/notif.mp3';
                    var audio = new Audio(src);
                    audio.play();
          }

          if(data.type == "text"){
            self.create_message(data.name+"::"+data.msg,data.uid,data.name);
          }

          if(data.type == "image"){
            self.create_file_message(data.msg,data.uid,self.name,data.data )
          }
          //self.message_id++;
          
        });
        self.peer_connection.on('open', () => {
          var payload = {
            type:"text",
            name:self.my_username,
            uid:self.my_peer_id,
            msg:self.my_username+"::connected"
          };
          self.peer_connection.send(payload);
         
        });
      });
    }
  },
  async created () {
    
  },
}
</script>
