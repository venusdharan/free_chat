<template>
  <v-row justify="center" align="start">
    <v-col cols="2">
      <v-text-field label="Connect Code" v-model="connect_id">
      </v-text-field>
      <v-btn block @click="connect_to_peer">Connect</v-btn>
      <v-divider></v-divider>
      <v-text-field label="My Connection Code" readonly style="margin-top:50px;" v-model="my_peer_id"></v-text-field>
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
          :rooms-loaded="roomsLoaded"
          :room-actions="roomActions"
          :menu-actions="menuActions"
          :room-message="roomMessage"
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
                    roomName: 'Room 1',
                    avatar: '',
                    unreadCount: 4,
                    index: 3,
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
                    users: [
                      {
                        _id: 1234,
                        username: 'John Doe',
                        avatar: '',
                        status: {
                          state: 'online',
                          lastChanged: 'today, 14:30'
                        }
                      },
                      {
                        _id: 4321,
                        username: 'John Snow',
                        avatar: '',
                        status: {
                          state: 'offline',
                          lastChanged: '14 July, 20:00'
                        }
                      }
                    ],
                    typingUsers: [ 4321 ]
                  }
      ],
			roomId: 1,
			startRooms: null,
			endRooms: null,
			roomsLoaded: true,
			loadingRooms: false,
			allUsers: [
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
      ],
      currentUserId: '6R0MijpK6M4AIrwaaCY2',
			loadingLastMessageByRoom: 0,
			roomsLoadedCount: false,
			selectedRoom: null,
			messagesPerPage: 20,
			messages: [
        {
    _id: 7890,
    content: 'message 1',
    senderId: 1234,
    username: 'John Doe',
    date: '13 November',
    timestamp: '10:20',
    system: false,
    saved: true,
    distributed: true,
    seen: true,
    disableActions: false,
    disableReactions: false,
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
    }
  }
      ],
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
			styles: { container: { borderRadius: '4px' } },
      peer_local:null,

      my_peer_id:null,
      connect_id:null,
    }
  },
  computed: {
		screenHeight() {
			return this.isDevice ? window.innerHeight + 'px' : 'calc(95vh - 80px)'
		}
	},
  methods: {
    connect_to_peer(){
      if(this.connect_id){
        var slf = this;
        console.log(slf.connect_id);
        this.peer_local = this.$peer.connect(slf.connect_id);
        this.$peer.on('open', function(id) {
          // Receive messages
          slf.$peer.on('data', function(data) {
            console.log('Received', data);
          });

          console.log(id);

          // Send messages
          slf.$peer.send('Hello!');
        });
      }
    },
    get_recieve(){
    var slf = this;
     this.peer_local = this.$peer.connect();
      this.$peer.on('open', function(id) {
        slf.my_peer_id = id;
      });
    }
  },
  async created () {
    this.get_recieve();
  },
}
</script>
