<template>
  <div>
    <div class="chats" v-if=" !inChat && hiddenMes">
      <div class="create">
        <button v-on:click="openModalWindow">
          Create
        </button>
      </div>
      <div class="logout">
        <button v-on:click="logout">
          Logout
        </button>
      </div>
      <div class="scroll" id="style-12">
        <div v-for="chat in chats" :key="chat.id">
          <button :id="chat.id" v-on:click="enterChat" :data-text="chat.name"
                  :class="{ 'chat-owner' : chat.owner.id === me.id }">
            {{ chat.name }}
          </button>
        </div>
      </div>
    </div>
    <div class="chats" v-if="inChat || !hiddenMes">
      <div>
        <p>
          {{ chat.name }}
        </p>
      </div>
      <div class="leaveButton">
        <button v-on:click="leaveChat">
          Leave
        </button>
      </div>
      <div class="deleteButton">
        <button v-on:click="deleteChat" v-if="chatOwner">
          Delete
        </button>
      </div>
      <div class="updateButton">
        <button v-on:click="openModalWindow2" v-if="chatOwner">
          Update
        </button>
      </div>
      <div class="messages" id="style-12">
        <div v-for="message in this.chat.lastMessages" :key="message.id">
          <div class="message" :class="{ 'owner-message' : message.author.id === me.id }">
            <div class="message-author">
              {{ message.author.username }}
            </div>
            <div class="message-text" id="style-11">
              {{ message.text }}
            </div>
            <div class="time">
              {{ message.timestamp.slice(11).split("").reverse().join("").slice(5).split("").reverse().join("") }}
            </div>
          </div>
        </div>
      </div>
      <div class="chatInput">
        <input type="text" v-model="message" placeholder="Enter message" required/>
        <button type="submit" v-on:click="createMessage">
          Send
        </button>
      </div>
      <div class="members" id="style-11">
        <div class="member" v-for="member in members" :key="member.id">
          {{ member.username }}
          <span class="span-me" v-if="member.id === me.id">(me)</span>
        </div>
      </div>
    </div>
    <div class="modals" v-if="!hiddenModal">
      <div class="centered-window">
        <button class="close-modal" v-on:click="closeModal">x</button>
        <div class="modal">
          <p>Enter new chat name:</p>
          <input type="text" placeholder="New chat name:" v-model="roomName" required minlength="1"/>
          <button class="submit-button" v-on:click="createChat">Create</button>
        </div>
      </div>
    </div>
    <div class="modals" v-if="!hiddenModal2">
      <div class="centered-window">
        <button class="close-modal" v-on:click="closeModal">x</button>
        <div class="modal">
          <p>Enter new chat name:</p>
          <input type="text" placeholder="Update chat name:" v-model="newRoomName" required minlength="1"/>
          <button class="submit-button" v-on:click="updateChat">Update</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import gql from "graphql-tag"

const rooms = gql`query rooms {
  chats: rooms {id, name, owner {id,username}, members {id, username}}
}`
const joinRoom = gql`mutation join($id:ID!) {
  joinRoom(roomId: $id) {id
    name
    timestamp
    lastMessages {
      text
      timestamp
      author {
        id
        username
      }
    }
    owner {
      id
      username
    }
    members {
      id
      username
    }
  }
}`
const leaveRoom = gql`mutation leave {
    leaveCurrentRoom {
      id
      name
    }
  }`
const createMessage = gql`mutation createMessage($message:String!) {
  createMessage(text: $message) {
    id
    timestamp
    author {id username }
    text
  }
}`
const userData = gql`  query me {
    me {
      id
      username
      timestamp
      rooms{
        id
        name
      }
      currentRoom {
        id
        name
        owner {
          id
          username
        }
        members{
            id
            username
          }
        lastMessages{
          id
          timestamp

          author{
            id
            username
          }
          text
        }
      }
    }
}`
const createdMessageSub = gql`subscription messageCreated {
  messageCreated {
    id
    timestamp
    author {
      id
      username
    }
    text
  }
}`
const createRoom = gql`mutation createRoom($roomName:String!) {
  createRoom(name: $roomName) {
    id
    name
    timestamp
    owner {
      id
      username
    }
  }
}`;
const updateRoom = gql`mutation updateRoom($name:String!,$id:ID!) {
    updateRoom(id: $id, name: $name) {
      id
      name
      timestamp
    }
  }`
const deleteRoom = gql`mutation deleteRoom($id:ID!) {
    deleteRoom(id: $id) {
      id
      name
      timestamp
    }
  }`
const createdRoomSub = gql`subscription roomCreated {
  roomCreated {
    id
    name
    timestamp
    owner {
      id
      username
    }
  }
}`
const updatedRoomSub = gql`subscription roomUpdated {
  roomUpdated {
    id
    name
    timestamp
    owner {
      id
      username
    }
    lastMessages {
      text
    }
    members {
      id
      username
    }
  }
}`
import {onLogout} from "../vue-apollo";

const deletedRoomSub = gql`subscription roomDeleted {
  roomDeleted {
    id
    name
    timestamp
    owner {
      id
      username
    }
  }
}`
const joinedUser = gql`subscription memberJoined {
  memberJoined {
    id
    username
  }
}`
const leftUser = gql`subscription memberLeft {
  memberLeft {
    id
    username
  }
}`

export default {
  data() {
    return {
      chat: {},
      chats: [],
      id: '',
      hiddenMes: true,
      message: '',
      inChat: false,
      me: {},
      members: [],
      messages: [],
      roomName: '',
      newRoomName: '',
      hiddenModal: true,
      hiddenModal2: true,
      chatOwner: false,
    };
  },
  name: "Chats",
  async created() {
    const me = await this.$apollo.query({
      fetchPolicy: "no-cache",
      query: userData,
    });
    this.me = me.data.me;
    if (this.me.currentRoom) {
      await this.loadLastChat(this.me);
    }
    const chats = await this.$apollo.query({
      query: rooms
    })
    this.chats = chats.data.chats;
  },
  methods: {
    async createChat() {
      this.hiddenMes = true;
      const newRoom = await this.$apollo.mutate({
        fetchPolicy: "no-cache",
        mutation: createRoom,
        variables: {
          roomName: this.roomName,
        },
      })
      return newRoom.data
    },
    async deleteChat() {
      await this.$apollo.mutate({
        fetchPolicy: "no-cache",
        mutation: deleteRoom,
        variables: {
          id: this.id,
        },
      });
      this.hiddenMes = true;
      this.me.currentRoom = null;
      this.chat = null;
    },
    async updateChat() {
      this.modalIsHidden2 = true
      const updatedRoom = await this.$apollo.mutate({
        fetchPolicy: "no-cache",
        mutation: updateRoom,
        variables: {
          id: this.id,
          name: this.newRoomName,
        },
      })
      console.log(updatedRoom.data)
      return updatedRoom.data
    },
    async createMessage() {
      await this.$apollo.mutate({
        fetchPolicy: "no-cache",
        mutation: createMessage,
        variables: {
          message: this.message
        },
      })
      this.message = ''
    },
    async scrollDown() {
      this.$nextTick(() => {
        const container = this.$el.querySelector(".messages");
        container.scrollTop = container.scrollHeight;
      });
    },
    async leaveChat() {
      this.hiddenMes = true;
      await this.$apollo.mutate({
        fetchPolicy: "no-cache",
        mutation: leaveRoom,
      })
    },
    async enterChat(event) {
      this.id = event.target.id
      this.chat = await this.joinRoom();
      this.hiddenMes = false;
      this.messages = this.chat.lastMessages;
      this.members = this.chat.members;
      await this.scrollDown();
      this.chatOwner = this.me.username === this.chat.owner.username;
    },
    async loadLastChat(me) {
      this.id = me.currentRoom.id;
      this.chat = me.currentRoom;
      this.messages = this.chat.lastMessages;
      this.members = this.chat.members;
      this.hiddenMes = false;
      await this.scrollDown();
      this.chatOwner = this.me.username === this.chat.owner.username;
    },
    async joinRoom() {
      const room = await this.$apollo.mutate({
        fetchPolicy: "no-cache",
        mutation: joinRoom,
        variables: {
          id: this.id
        },
      })
      return room.data.joinRoom
    },
    async openModalWindow() {
      this.hiddenModal = false
    },
    async openModalWindow2() {
      this.hiddenModal2 = false
    },
    async closeModal() {
      this.hiddenModal = true
      this.hiddenModal2 = true
    },
    async logout() {
      await onLogout(this.$apollo.provider.defaultClient);
      await this.$apollo.provider.defaultClient.resetStore();
      await this.$router.push('/')
    },
  },
  apollo: {
    $subscribe: {
      messageCreation: {
        query: createdMessageSub,
        result({data}) {
          console.log(data.messageCreated)
          this.messages.push(data.messageCreated)
          this.scrollDown()
        },
      },
      roomCreation: {
        query: createdRoomSub,
        result({data}) {
          this.chats.push(data.roomCreated)
        },
      },
      roomChange: {
        query: updatedRoomSub,
        result({data}) {
          this.chats = this.chats.map(chat => {
            if (chat.id === data.roomUpdated.id) {
              this.chat.name = data.roomUpdated.name
            }
            return chat
          })
        },
      },
      roomDeletion: {
        query: deletedRoomSub,
        result({data}) {
          this.hiddenMes = true
          let specificIndex = -1
          this.chats.forEach((chat, roomIndex) =>
              chat.id === data.roomDeleted.id ? (specificIndex = roomIndex) : ""
          );
          this.chats.splice(specificIndex, 1)
        },
      },
      joinedUser: {
        query: joinedUser,
        result({data}) {
          console.log(data)
          this.members.push(data.memberJoined)
        },
      },
      leftUser: {
        query: leftUser,
        result({data}) {
          console.log(data)
          let specificIndex = -1;
          this.members.forEach((member, memberIndex) =>
              member.id === data.memberLeft.id ? (specificIndex = memberIndex) : ""
          );
          this.members.splice(specificIndex, 1);
        },
      },
    },
  },
}
</script>

<style scoped>
.chats {
  height: 80vh;
  margin-left: 10vw;
  margin-right: 10vw;
  background-color: rgba(201, 178, 178, 0.7);
  box-shadow: 20px 20px 50px rgba(0, 0, 0, .5);
  backdrop-filter: blur(10px);
  border-radius: 4%;
  overflow: auto;
}

.scroll {
  width: 20vw;
  height: 75vh;
  overflow-y: auto;
  margin: auto;
  margin-top: 3vh;
}

.scroll button {
  width: 95%;
  height: 4vh;
  margin-bottom: 5%;
  border-radius: 10px;
}

#style-12::-webkit-scrollbar-track {
  border-radius: 10px;
  -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.9);
  background-color: #444444;
}

#style-12::-webkit-scrollbar {
  border-radius: 10px;
  width: 12px;
  background-color: #F5F5F5;
}

#style-12::-webkit-scrollbar-thumb {
  border-radius: 10px;
  background-color: #D62929;
  background-image: -webkit-linear-gradient(90deg,
  transparent,
  rgba(0, 0, 0, 0.4) 50%,
  transparent,
  transparent)
}

.create button {
  *margin: unset;
  float: left;
  margin-left: 15vw;
  width: 7vw;
  height: 4vh;
  border-radius: 10px;
  font-size: 20px;
  margin-top: 2vh;
}

.logout button {
  justify-content: center;
  *margin: unset;
  float: right;
  margin-right: 15vw;
  width: 7vw;
  height: 4vh;
  border-radius: 10px;
  font-size: 20px;
  margin-top: 2vh;
}

.messages {
  margin: auto;
  margin-top: 3vh;
  width: 37vw;
  height: 62vh;
  overflow-y: auto;
  overflow-x: hidden;
}

.chatInput {
  display: flex;
  justify-content: center;
  margin-top: 3vw;
}

.chatInput input {
  width: 30vw;
  height: 4vh;
  border-radius: 10px;
  background-color: #443c3c;
  font-family: 'Handlee', cursive;
  font-size: 28px;
  font-weight: 600;
  color: #ebebeb;
  text-align: center;
  border: solid;
}

.chatInput input::placeholder {
  text-align: center;
  color: #ebebeb;
  font-family: 'Handlee', cursive;
  font-size: 28px;
  font-weight: 600;
}

.chatInput button {
  width: 6vw;
  font-size: 28px;
  border-radius: 10px;
  margin: 0;
  margin-left: 1%;
}

.message-author {
  font-family: cursive;
  font-size: 20px;
  color: #e23734;
}

.message {
  width: 50%;
  float: left;
  background-color: #3c3c3c;
  border-radius: 10px;
  border: 2px solid #ebebeb;
  margin-bottom: 4%;
  padding: 10px 0px 10px 20px;
  background: linear-gradient(0.25turn, #443c3c, #791e1e);
}

.owner-message {
  float: right;
  background: linear-gradient(0.25turn, #16222A, #3A6073);
  margin-right: 2%;
}

.message-text {
  font-size: 18px;
  height: 2.5vh;
  width: 18vw;
  overflow-y: auto;
  overflow-x: hidden;
  text-align: left;
  word-wrap: break-word;
  color: #ebebeb;
}

.time {
  font-size: 12px;
}

#style-11::-webkit-scrollbar-track {
  border-radius: 10px;
  -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.9);
  background-color: #444444;
}

#style-11::-webkit-scrollbar {
  border-radius: 10px;
  width: 12px;
  background-color: #F5F5F5;
}

#style-11::-webkit-scrollbar-thumb {
  border-radius: 10px;
  background-color: #b68080;
  background-image: -webkit-linear-gradient(90deg,
  transparent,
  rgba(0, 0, 0, 0.4) 50%,
  transparent,
  transparent)
}

.members {
  position: absolute;
  right: 5%;
  top: 4%;
  height: 62vh;
  font-size: 30px;
  overflow-y: auto;
}

.modals {
  position: absolute;
  left: 0;
  top: 0;
  z-index: 10;
  width: 100%;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
}

.centered-window {
  position: relative;
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 25px;
  width: 40%;
  height: 22vh;
  text-align: center;
}
.centered-window input{
  width: 20vw;
  height: 3vh;
  border-radius: 10px;
  background-color: #443c3c;
  font-family: 'Handlee', cursive;
  font-size: 20px;
  font-weight: 600;
  color: #ebebeb;
  text-align: center;
  border: solid;
}
.modal{
  margin-top: 7%
}

.close-modal{
  position: absolute;
  right: 10px;
  width: 3%;
  height: 10%;
  border-radius: 90%;
  background-color: #c4221f;
}

.chat-owner {
  border: 2px solid #c4221f;
  box-shadow: 0 0 10px #c4221f;
}
.chats div p{
  position: absolute;
  top: 6%;
  left: 3%;
  font-size: 25px;
  color: #c4221f;
  text-shadow: 0 0 10px #c4221f;
}

.submit-button {
  margin-top: 3vh;
  width: 7vw;
  height: 4vh;
  font-family: 'Handlee', cursive;
  font-size: 28px;
  font-weight: 600;
  border-radius: 10px;
}

.leaveButton button {
  position: absolute;
  margin-top: 8%;
  margin-left: 3%;
  float: left;
  width: 7vw;
  height: 4vh;
  font-family: 'Handlee', cursive;
  font-size: 28px;
  font-weight: 600;
  border-radius: 10px;
}

.deleteButton button {
  position: absolute;
  margin-top: 18%;
  margin-left: 3%;
  float: left;
  width: 7vw;
  height: 4vh;
  font-family: 'Handlee', cursive;
  font-size: 28px;
  font-weight: 600;
  border-radius: 10px;
}

.updateButton button {
  position: absolute;
  margin-top: 13%;
  margin-left: 3%;
  float: left;
  width: 7vw;
  height: 4vh;
  font-family: 'Handlee', cursive;
  font-size: 28px;
  font-weight: 600;
  border-radius: 10px;
}



</style>