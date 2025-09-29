<script setup>
import { GoogleGenerativeAI } from "@google/generative-ai";
import { ref, onMounted } from "vue";
import { marked } from "marked";

const message = ref("");
const conversations = ref([]); // all chats
const activeChat = ref(null);  // currently open chat
const error = ref(null);
const menu = ref(false);

const GEMINI_API_KEY = "AIzaSyCUqUFL4HPEDkH_2xm16qg3WVLjV2gqcnE";
const genAI = new GoogleGenerativeAI(GEMINI_API_KEY);

const model = genAI.getGenerativeModel({ model: "gemini-2.5-flash" });
let chatSession = null;

// ✅ Load saved chats
onMounted(() => {
  const saved = localStorage.getItem("conversations");
  if (saved) {
    conversations.value = JSON.parse(saved);
  }
});

function startNewChat() {
  const newChat = {
    id: Date.now(),
    title: "New Chat",
    messages: []
  };
  conversations.value.unshift(newChat);
  activeChat.value = newChat;
  chatSession = model.startChat({ history: [] });
  saveChats();
}

async function sendMessage() {
  if (!message.value.trim()) return;

  try {
    // ✅ If no active chat, auto-create one
    if (!activeChat.value) {
      startNewChat();
    }

    if (!chatSession) {
      chatSession = model.startChat({ history: [] });
    }

    // Add user message first
    activeChat.value.messages.push({ role: "user", text: message.value });

    // Send to Gemini
    const result = await chatSession.sendMessage(message.value);
    const text = result.response.text();

    // Add AI reply
    activeChat.value.messages.push({ role: "model", text });

    // Update title if still default
    if (activeChat.value.title === "New Chat") {
      activeChat.value.title = message.value.split(" ").slice(0, 5).join(" ") + '...';
    }

    message.value = "";
    saveChats();
  } catch (err) {
    error.value = "⚠️ " + err.message;
  }
}


function selectChat(chat) {
  activeChat.value = chat;
}
function unSelectChat(chat) {
  activeChat.value = '';
}

function deleteChat(chatId) {
  conversations.value = conversations.value.filter(c => c.id !== chatId);
  if (activeChat.value?.id === chatId) {
    activeChat.value = conversations.value[0] || null;
  }
  saveChats();
}

function saveChats() {
  localStorage.setItem("conversations", JSON.stringify(conversations.value));
}



const toggleMenu = () => menu.value = !menu.value;
const showoptions = (chatId) => {
  const el = document.getElementById(chatId);
  if (!el) return;

  const currentDeleteMenu = el.nextElementSibling;

  const wasCurrentMenuOpen = currentDeleteMenu &&
    currentDeleteMenu.classList.contains('deleteChat') &&
    currentDeleteMenu.style.display === 'block';

  const allDeleteMenus = document.querySelectorAll('.deleteChat');
  allDeleteMenus.forEach(menu => {
    menu.style.display = 'none';
  });

  if (currentDeleteMenu && currentDeleteMenu.classList.contains('deleteChat')) {
    if (!wasCurrentMenuOpen) {
      currentDeleteMenu.style.display = 'block';
    }
  }
};

</script>
<template>
  <main class="flex overflow-hidden">
    <aside :class="menu ? 'bg-gray-50 w-65 p-3 pt-2' : 'md:w-15 pt-2'"
      class="flex flex-col border-r border-gray-100 h-screen transition-all pb-20 z-20 absolute md:relative">
      <div class='flex items-center w-full group' :class="menu ? ' justify-between' : ' justify-center'">
        <div @click="unSelectChat" class="cursor-pointer p-2 hover:bg-gray-100 rounded-xl  group-hover:hidden"
          :class="menu ? '!block' : ' hidden md:block'">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="currentColor" xmlns="http://www.w3.org/2000/svg"
            class="w-6 h-6">
            <path
              d="M11.2475 18.25C10.6975 18.25 10.175 18.1455 9.67999 17.9365C9.18499 17.7275 8.74499 17.436 8.35999 17.062C7.94199 17.205 7.50749 17.2765 7.05649 17.2765C6.31949 17.2765 5.63749 17.095 5.01049 16.732C4.38349 16.369 3.87749 15.874 3.49249 15.247C3.11849 14.62 2.93149 13.9215 2.93149 13.1515C2.93149 12.8325 2.97549 12.486 3.06349 12.112C2.62349 11.705 2.28249 11.2375 2.04049 10.7095C1.79849 10.1705 1.67749 9.6095 1.67749 9.0265C1.67749 8.4325 1.80399 7.8605 2.05699 7.3105C2.30999 6.7605 2.66199 6.2875 3.11299 5.8915C3.57499 5.4845 4.10849 5.204 4.71349 5.05C4.83449 4.423 5.08749 3.862 5.47249 3.367C5.86849 2.861 6.35249 2.465 6.92449 2.179C7.49649 1.893 8.10699 1.75 8.75599 1.75C9.30599 1.75 9.82849 1.8545 10.3235 2.0635C10.8185 2.2725 11.2585 2.564 11.6435 2.938C12.0615 2.795 12.496 2.7235 12.947 2.7235C13.684 2.7235 14.366 2.905 14.993 3.268C15.62 3.631 16.1205 4.126 16.4945 4.753C16.8795 5.38 17.072 6.0785 17.072 6.8485C17.072 7.1675 17.028 7.514 16.94 7.888C17.38 8.295 17.721 8.768 17.963 9.307C18.205 9.835 18.326 10.3905 18.326 10.9735C18.326 11.5675 18.1995 12.1395 17.9465 12.6895C17.6935 13.2395 17.336 13.718 16.874 14.125C16.423 14.521 15.895 14.796 15.29 14.95C15.169 15.577 14.9105 16.138 14.5145 16.633C14.1295 17.139 13.651 17.535 13.079 17.821C12.507 18.107 11.8965 18.25 11.2475 18.25ZM7.17199 16.1875C7.72199 16.1875 8.20049 16.072 8.60749 15.841L11.7095 14.059C11.8195 13.982 11.8745 13.8775 11.8745 13.7455V12.3265L7.88149 14.62C7.63949 14.763 7.39749 14.763 7.15549 14.62L4.03699 12.8215C4.03699 12.8545 4.03149 12.893 4.02049 12.937C4.02049 12.981 4.02049 13.047 4.02049 13.135C4.02049 13.696 4.15249 14.213 4.41649 14.686C4.69149 15.148 5.07099 15.511 5.55499 15.775C6.03899 16.05 6.57799 16.1875 7.17199 16.1875ZM7.33699 13.498C7.40299 13.531 7.46349 13.5475 7.51849 13.5475C7.57349 13.5475 7.62849 13.531 7.68349 13.498L8.92099 12.7885L4.94449 10.4785C4.70249 10.3355 4.58149 10.121 4.58149 9.835V6.2545C4.03149 6.4965 3.59149 6.8705 3.26149 7.3765C2.93149 7.8715 2.76649 8.4215 2.76649 9.0265C2.76649 9.5655 2.90399 10.0825 3.17899 10.5775C3.45399 11.0725 3.81149 11.4465 4.25149 11.6995L7.33699 13.498ZM11.2475 17.161C11.8305 17.161 12.3585 17.029 12.8315 16.765C13.3045 16.501 13.6785 16.138 13.9535 15.676C14.2285 15.214 14.366 14.697 14.366 14.125V10.561C14.366 10.429 14.311 10.33 14.201 10.264L12.947 9.538V14.1415C12.947 14.4275 12.826 14.642 12.584 14.785L9.46549 16.5835C10.0045 16.9685 10.5985 17.161 11.2475 17.161ZM11.8745 11.122V8.878L10.01 7.822L8.12899 8.878V11.122L10.01 12.178L11.8745 11.122ZM7.05649 5.8585C7.05649 5.5725 7.17749 5.358 7.41949 5.215L10.538 3.4165C9.99899 3.0315 9.40499 2.839 8.75599 2.839C8.17299 2.839 7.64499 2.971 7.17199 3.235C6.69899 3.499 6.32499 3.862 6.04999 4.324C5.78599 4.786 5.65399 5.303 5.65399 5.875V9.4225C5.65399 9.5545 5.70899 9.659 5.81899 9.736L7.05649 10.462V5.8585ZM15.4385 13.7455C15.9885 13.5035 16.423 13.1295 16.742 12.6235C17.072 12.1175 17.237 11.5675 17.237 10.9735C17.237 10.4345 17.0995 9.9175 16.8245 9.4225C16.5495 8.9275 16.192 8.5535 15.752 8.3005L12.6665 6.5185C12.6005 6.4745 12.54 6.458 12.485 6.469C12.43 6.469 12.375 6.4855 12.32 6.5185L11.0825 7.2115L15.0755 9.538C15.1965 9.604 15.2845 9.692 15.3395 9.802C15.4055 9.901 15.4385 10.022 15.4385 10.165V13.7455ZM12.122 5.3635C12.364 5.2095 12.606 5.2095 12.848 5.3635L15.983 7.195C15.983 7.118 15.983 7.019 15.983 6.898C15.983 6.37 15.851 5.8695 15.587 5.3965C15.334 4.9125 14.9655 4.5275 14.4815 4.2415C14.0085 3.9555 13.4585 3.8125 12.8315 3.8125C12.2815 3.8125 11.803 3.928 11.396 4.159L8.29399 5.941C8.18399 6.018 8.12899 6.1225 8.12899 6.2545V7.6735L12.122 5.3635Z">
            </path>
          </svg>
        </div>
        <div class="cursor-e-resize p-2 hover:bg-gray-100 rounded-xl hidden group-hover:block" @click="toggleMenu"
          :class="menu ? '!block' : ''">
          <svg :class="menu ? 'fill-gray-400' : ''" width="20" height="20" viewBox="0 0 20 20" fill="currentColor"
            xmlns="http://www.w3.org/2000/svg" data-rtl-flip="" class="w-6 h-6 hidden md:block">
            <path
              d="M6.83496 3.99992C6.38353 4.00411 6.01421 4.0122 5.69824 4.03801C5.31232 4.06954 5.03904 4.12266 4.82227 4.20012L4.62207 4.28606C4.18264 4.50996 3.81498 4.85035 3.55859 5.26848L3.45605 5.45207C3.33013 5.69922 3.25006 6.01354 3.20801 6.52824C3.16533 7.05065 3.16504 7.71885 3.16504 8.66301V11.3271C3.16504 12.2712 3.16533 12.9394 3.20801 13.4618C3.25006 13.9766 3.33013 14.2909 3.45605 14.538L3.55859 14.7216C3.81498 15.1397 4.18266 15.4801 4.62207 15.704L4.82227 15.79C5.03904 15.8674 5.31234 15.9205 5.69824 15.9521C6.01398 15.9779 6.383 15.986 6.83398 15.9902L6.83496 3.99992ZM18.165 11.3271C18.165 12.2493 18.1653 12.9811 18.1172 13.5702C18.0745 14.0924 17.9916 14.5472 17.8125 14.9648L17.7295 15.1415C17.394 15.8 16.8834 16.3511 16.2568 16.7353L15.9814 16.8896C15.5157 17.1268 15.0069 17.2285 14.4102 17.2773C13.821 17.3254 13.0893 17.3251 12.167 17.3251H7.83301C6.91071 17.3251 6.17898 17.3254 5.58984 17.2773C5.06757 17.2346 4.61294 17.1508 4.19531 16.9716L4.01855 16.8896C3.36014 16.5541 2.80898 16.0434 2.4248 15.4169L2.27051 15.1415C2.03328 14.6758 1.93158 14.167 1.88281 13.5702C1.83468 12.9811 1.83496 12.2493 1.83496 11.3271V8.66301C1.83496 7.74072 1.83468 7.00898 1.88281 6.41985C1.93157 5.82309 2.03329 5.31432 2.27051 4.84856L2.4248 4.57317C2.80898 3.94666 3.36012 3.436 4.01855 3.10051L4.19531 3.0175C4.61285 2.83843 5.06771 2.75548 5.58984 2.71281C6.17898 2.66468 6.91071 2.66496 7.83301 2.66496H12.167C13.0893 2.66496 13.821 2.66468 14.4102 2.71281C15.0069 2.76157 15.5157 2.86329 15.9814 3.10051L16.2568 3.25481C16.8833 3.63898 17.394 4.19012 17.7295 4.84856L17.8125 5.02531C17.9916 5.44285 18.0745 5.89771 18.1172 6.41985C18.1653 7.00898 18.165 7.74072 18.165 8.66301V11.3271ZM8.16406 15.995H12.167C13.1112 15.995 13.7794 15.9947 14.3018 15.9521C14.8164 15.91 15.1308 15.8299 15.3779 15.704L15.5615 15.6015C15.9797 15.3451 16.32 14.9774 16.5439 14.538L16.6299 14.3378C16.7074 14.121 16.7605 13.8478 16.792 13.4618C16.8347 12.9394 16.835 12.2712 16.835 11.3271V8.66301C16.835 7.71885 16.8347 7.05065 16.792 6.52824C16.7605 6.14232 16.7073 5.86904 16.6299 5.65227L16.5439 5.45207C16.32 5.01264 15.9796 4.64498 15.5615 4.3886L15.3779 4.28606C15.1308 4.16013 14.8165 4.08006 14.3018 4.03801C13.7794 3.99533 13.1112 3.99504 12.167 3.99504H8.16406C8.16407 3.99667 8.16504 3.99829 8.16504 3.99992L8.16406 15.995Z">
            </path>
          </svg>
          <svg width="20" height="20" viewBox="0 0 20 20" fill="currentColor" xmlns="http://www.w3.org/2000/svg"
            class="icon md:hidden">
            <path
              d="M14.2548 4.75488C14.5282 4.48152 14.9717 4.48152 15.2451 4.75488C15.5184 5.02825 15.5184 5.47175 15.2451 5.74512L10.9902 10L15.2451 14.2549L15.3349 14.3652C15.514 14.6369 15.4841 15.006 15.2451 15.2451C15.006 15.4842 14.6368 15.5141 14.3652 15.335L14.2548 15.2451L9.99995 10.9902L5.74506 15.2451C5.4717 15.5185 5.0282 15.5185 4.75483 15.2451C4.48146 14.9718 4.48146 14.5282 4.75483 14.2549L9.00971 10L4.75483 5.74512L4.66499 5.63477C4.48589 5.3631 4.51575 4.99396 4.75483 4.75488C4.99391 4.51581 5.36305 4.48594 5.63471 4.66504L5.74506 4.75488L9.99995 9.00977L14.2548 4.75488Z">
            </path>
          </svg>
        </div>
      </div>
      <div :class="menu ? 'items-start' : 'items-center'"
        class="flex w-full flex-col justify-center gap-3 mt-4 relative">
        <button :class="menu ? 'justify-start  w-full ' : 'justify-center hidden md:block'" @click="startNewChat"
          class="flex items-center gap-2 p-2 hover:bg-gray-100 rounded-xl cursor-pointer transition-all duration-200 ease-in-out">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="currentColor" xmlns="http://www.w3.org/2000/svg"
            class="icon">
            <path
              d="M2.6687 11.333V8.66699C2.6687 7.74455 2.66841 7.01205 2.71655 6.42285C2.76533 5.82612 2.86699 5.31731 3.10425 4.85156L3.25854 4.57617C3.64272 3.94975 4.19392 3.43995 4.85229 3.10449L5.02905 3.02149C5.44666 2.84233 5.90133 2.75849 6.42358 2.71582C7.01272 2.66769 7.74445 2.66797 8.66675 2.66797H9.16675C9.53393 2.66797 9.83165 2.96586 9.83179 3.33301C9.83179 3.70028 9.53402 3.99805 9.16675 3.99805H8.66675C7.7226 3.99805 7.05438 3.99834 6.53198 4.04102C6.14611 4.07254 5.87277 4.12568 5.65601 4.20313L5.45581 4.28906C5.01645 4.51293 4.64872 4.85345 4.39233 5.27149L4.28979 5.45508C4.16388 5.7022 4.08381 6.01663 4.04175 6.53125C3.99906 7.05373 3.99878 7.7226 3.99878 8.66699V11.333C3.99878 12.2774 3.99906 12.9463 4.04175 13.4688C4.08381 13.9833 4.16389 14.2978 4.28979 14.5449L4.39233 14.7285C4.64871 15.1465 5.01648 15.4871 5.45581 15.7109L5.65601 15.7969C5.87276 15.8743 6.14614 15.9265 6.53198 15.958C7.05439 16.0007 7.72256 16.002 8.66675 16.002H11.3337C12.2779 16.002 12.9461 16.0007 13.4685 15.958C13.9829 15.916 14.2976 15.8367 14.5447 15.7109L14.7292 15.6074C15.147 15.3511 15.4879 14.9841 15.7117 14.5449L15.7976 14.3447C15.8751 14.128 15.9272 13.8546 15.9587 13.4688C16.0014 12.9463 16.0017 12.2774 16.0017 11.333V10.833C16.0018 10.466 16.2997 10.1681 16.6667 10.168C17.0339 10.168 17.3316 10.4659 17.3318 10.833V11.333C17.3318 12.2555 17.3331 12.9879 17.2849 13.5771C17.2422 14.0993 17.1584 14.5541 16.9792 14.9717L16.8962 15.1484C16.5609 15.8066 16.0507 16.3571 15.4246 16.7412L15.1492 16.8955C14.6833 17.1329 14.1739 17.2354 13.5769 17.2842C12.9878 17.3323 12.256 17.332 11.3337 17.332H8.66675C7.74446 17.332 7.01271 17.3323 6.42358 17.2842C5.90135 17.2415 5.44665 17.1577 5.02905 16.9785L4.85229 16.8955C4.19396 16.5601 3.64271 16.0502 3.25854 15.4238L3.10425 15.1484C2.86697 14.6827 2.76534 14.1739 2.71655 13.5771C2.66841 12.9879 2.6687 12.2555 2.6687 11.333ZM13.4646 3.11328C14.4201 2.334 15.8288 2.38969 16.7195 3.28027L16.8865 3.46485C17.6141 4.35685 17.6143 5.64423 16.8865 6.53613L16.7195 6.7207L11.6726 11.7686C11.1373 12.3039 10.4624 12.6746 9.72827 12.8408L9.41089 12.8994L7.59351 13.1582C7.38637 13.1877 7.17701 13.1187 7.02905 12.9707C6.88112 12.8227 6.81199 12.6134 6.84155 12.4063L7.10132 10.5898L7.15991 10.2715C7.3262 9.53749 7.69692 8.86241 8.23218 8.32715L13.2791 3.28027L13.4646 3.11328ZM15.7791 4.2207C15.3753 3.81702 14.7366 3.79124 14.3035 4.14453L14.2195 4.2207L9.17261 9.26856C8.81541 9.62578 8.56774 10.0756 8.45679 10.5654L8.41772 10.7773L8.28296 11.7158L9.22241 11.582L9.43433 11.543C9.92426 11.432 10.3749 11.1844 10.7322 10.8271L15.7791 5.78027L15.8552 5.69629C16.185 5.29194 16.1852 4.708 15.8552 4.30371L15.7791 4.2207Z">
            </path>
          </svg>
          <span v-if="menu">New Chat</span>
        </button>
        <span v-if="menu" class="text-gray-400 text-sm px-2">Chats</span>
        <div v-if="menu" class="flex flex-col gap-2 w-full">
          <div v-for="chat in conversations" :key="chat.id"
            :class="activeChat && activeChat.id == chat.id ? 'bg-gray-100' : ''"
            class="flex justify-between items-center p-2 hover:bg-gray-100 rounded-xl cursor-pointer "
            @click="selectChat(chat)">
            <span class="truncate w-36 ">{{ chat.title }}</span>
            <div class="relative">
              <span @click="showoptions(chat.id)" :id="chat.id">
                <svg width="20" height="20" viewBox="0 0 20 20" fill="currentColor" xmlns="http://www.w3.org/2000/svg"
                  class="icon" aria-hidden="true">
                  <path
                    d="M15.498 8.50159C16.3254 8.50159 16.9959 9.17228 16.9961 9.99963C16.9961 10.8271 16.3256 11.4987 15.498 11.4987C14.6705 11.4987 14 10.8271 14 9.99963C14.0002 9.17228 14.6706 8.50159 15.498 8.50159Z">
                  </path>
                  <path
                    d="M4.49805 8.50159C5.32544 8.50159 5.99689 9.17228 5.99707 9.99963C5.99707 10.8271 5.32555 11.4987 4.49805 11.4987C3.67069 11.4985 3 10.827 3 9.99963C3.00018 9.17239 3.6708 8.50176 4.49805 8.50159Z">
                  </path>
                  <path
                    d="M10.0003 8.50159C10.8276 8.50176 11.4982 9.17239 11.4984 9.99963C11.4984 10.827 10.8277 11.4985 10.0003 11.4987C9.17283 11.4987 8.50131 10.8271 8.50131 9.99963C8.50149 9.17228 9.17294 8.50159 10.0003 8.50159Z">
                  </path>
                </svg>
              </span>
              <div
                class="options z-10 bg-white p-1 absolute shadow border border-gray-200 rounded-2xl hidden deleteChat">
                <button @click.stop="deleteChat(chat.id)"
                  class="text-red-500 text-sm flex gap-2 items-center flex-start hover:bg-red-50 px-[10px] py-[7px] rounded-xl cursor-pointer pr-10">
                  <span><svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"
                      class="stroke-red-500 w-5 h-5">
                      <g id="SVGRepo_bgCarrier" stroke-width="0"></g>
                      <g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g>
                      <g id="SVGRepo_iconCarrier">
                        <path d="M10 12V17" stroke="#fb2c36 " stroke-width="2" stroke-linecap="round"
                          stroke-linejoin="round"></path>
                        <path d="M14 12V17" stroke="#fb2c36 " stroke-width="2" stroke-linecap="round"
                          stroke-linejoin="round"></path>
                        <path d="M4 7H20" stroke="#fb2c36 " stroke-width="2" stroke-linecap="round"
                          stroke-linejoin="round">
                        </path>
                        <path d="M6 10V18C6 19.6569 7.34315 21 9 21H15C16.6569 21 18 19.6569 18 18V10" stroke="#fb2c36 "
                          stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path>
                        <path d="M9 5C9 3.89543 9.89543 3 11 3H13C14.1046 3 15 3.89543 15 5V7H9V5Z" stroke="#fb2c36 "
                          stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path>
                      </g>
                    </svg>
                  </span>
                  <span class="text-sm z-20">Delete</span>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div :class="menu ? 'border-t bg-gray-50' : ''"
        class="fixed md:block hidden bottom-0 px-2 py-3 left-3 border-gray-200 w-57">
        <button class="flex items-center gap-2  text-sm">
          <span class="w-6 h-6 bg-blue-400 text-white rounded-full flex justify-center items-center">KA</span>
          <div v-if="menu" class="flex flex-col items-start">
            <span>Kaleemullah Ahsan</span>
            <span class="text-gray-400 ">Free</span>
          </div>
        </button>
      </div>
    </aside>
    <!-- Chat Window -->
    <div class="flex flex-1 flex-col m-auto justify-center items-center h-screen overflow-y-auto relative px-5">
      <div class="absolute left-3 top-3 flex items-center">
        <span class="hover:bg-gray-100 cursor-pointer rounded-md text-xl md:hidden" @click="toggleMenu">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="currentColor" xmlns="http://www.w3.org/2000/svg"
            data-rtl-flip="" class="w-7 h-7 fill-gray-500">
            <path
              d="M11.6663 12.6686L11.801 12.6823C12.1038 12.7445 12.3313 13.0125 12.3313 13.3337C12.3311 13.6547 12.1038 13.9229 11.801 13.985L11.6663 13.9987H3.33325C2.96609 13.9987 2.66839 13.7008 2.66821 13.3337C2.66821 12.9664 2.96598 12.6686 3.33325 12.6686H11.6663ZM16.6663 6.00163L16.801 6.0153C17.1038 6.07747 17.3313 6.34546 17.3313 6.66667C17.3313 6.98788 17.1038 7.25586 16.801 7.31803L16.6663 7.33171H3.33325C2.96598 7.33171 2.66821 7.03394 2.66821 6.66667C2.66821 6.2994 2.96598 6.00163 3.33325 6.00163H16.6663Z">
            </path>
          </svg>
        </span>
        <span class="py-1 px-3 hover:bg-gray-100 cursor-pointer rounded-md text-xl">ChatGPT</span>
      </div>
      <div v-if="!activeChat || activeChat.messages.length < 1"
        class=" flex items-center justify-center max-w-220 m-auto">
        <p class="md:text-3xl text-2xl">What can I help with?</p>
      </div>
      <div v-else class="pt-16  m-auto w-full h-full max-w-220">
        <div v-for="(m, i) in activeChat.messages" :key="i"
          :class="m.role === 'user' ? 'text-right' : 'text-left model'" class="mb-5">
          <div
            :class="m.role === 'user' ? 'bg-gray-100 inline-block px-4 py-2 rounded-3xl' : 'inline-block py-2 rounded-xl'"
            v-html="marked(m.text)">
          </div>
        </div>
      </div>
      <!-- Input -->
      <div
        class="fixed textInput w-full lg:max-w-220 md:max-w-130 md:left-auto md:right-auto left-2 right-2 bottom-0 bg-white  m-auto flex justify-center flex-col items-center">
        <div
          class="m-auto flex justify-center border bg-white border-gray-100 shadow gap-1 md:p-2 p-1 items-center rounded-full w-full ">
          <button class="cursor-not-allowed hover:bg-gray-50 rounded-full p-2">
            <svg width="20" height="20" viewBox="0 0 20 20" fill="currentColor" xmlns="http://www.w3.org/2000/svg"
              class="icon">
              <path
                d="M9.33496 16.5V10.665H3.5C3.13273 10.665 2.83496 10.3673 2.83496 10C2.83496 9.63273 3.13273 9.33496 3.5 9.33496H9.33496V3.5C9.33496 3.13273 9.63273 2.83496 10 2.83496C10.3673 2.83496 10.665 3.13273 10.665 3.5V9.33496H16.5L16.6338 9.34863C16.9369 9.41057 17.165 9.67857 17.165 10C17.165 10.3214 16.9369 10.5894 16.6338 10.6514L16.5 10.665H10.665V16.5C10.665 16.8673 10.3673 17.165 10 17.165C9.63273 17.165 9.33496 16.8673 9.33496 16.5Z">
              </path>
            </svg>
          </button>
          <textarea v-model="message" rows="1" class="flex-1  resize-none outline-none rounded p-2"
            placeholder="Ask anything"></textarea>
          <button class="cursor-not-allowed hover:bg-gray-50 rounded-full p-2">
            <svg width="20" height="20" viewBox="0 0 20 20" fill="currentColor" xmlns="http://www.w3.org/2000/svg"
              aria-label="" class="icon" font-size="inherit">
              <path
                d="M15.7806 10.1963C16.1326 10.3011 16.3336 10.6714 16.2288 11.0234L16.1487 11.2725C15.3429 13.6262 13.2236 15.3697 10.6644 15.6299L10.6653 16.835H12.0833L12.2171 16.8486C12.5202 16.9106 12.7484 17.1786 12.7484 17.5C12.7484 17.8214 12.5202 18.0894 12.2171 18.1514L12.0833 18.165H7.91632C7.5492 18.1649 7.25128 17.8672 7.25128 17.5C7.25128 17.1328 7.5492 16.8351 7.91632 16.835H9.33527L9.33429 15.6299C6.775 15.3697 4.6558 13.6262 3.84992 11.2725L3.76984 11.0234L3.74445 10.8906C3.71751 10.5825 3.91011 10.2879 4.21808 10.1963C4.52615 10.1047 4.84769 10.2466 4.99347 10.5195L5.04523 10.6436L5.10871 10.8418C5.8047 12.8745 7.73211 14.335 9.99933 14.335C12.3396 14.3349 14.3179 12.7789 14.9534 10.6436L15.0052 10.5195C15.151 10.2466 15.4725 10.1046 15.7806 10.1963ZM12.2513 5.41699C12.2513 4.17354 11.2437 3.16521 10.0003 3.16504C8.75675 3.16504 7.74835 4.17343 7.74835 5.41699V9.16699C7.74853 10.4104 8.75685 11.418 10.0003 11.418C11.2436 11.4178 12.2511 10.4103 12.2513 9.16699V5.41699ZM13.5814 9.16699C13.5812 11.1448 11.9781 12.7479 10.0003 12.748C8.02232 12.748 6.41845 11.1449 6.41828 9.16699V5.41699C6.41828 3.43889 8.02221 1.83496 10.0003 1.83496C11.9783 1.83514 13.5814 3.439 13.5814 5.41699V9.16699Z">
              </path>
            </svg>
          </button>
          <button @click="sendMessage" :class="!message ? 'opacity-50 !cursor-not-allowed' : ''"
            class="cursor-pointer bg-black rounded-full p-2">
            <svg width="20" height="20" viewBox="0 0 20 20" fill="white" xmlns="http://www.w3.org/2000/svg"
              class="icon">
              <path
                d="M8.99992 16V6.41407L5.70696 9.70704C5.31643 10.0976 4.68342 10.0976 4.29289 9.70704C3.90237 9.31652 3.90237 8.6835 4.29289 8.29298L9.29289 3.29298L9.36907 3.22462C9.76184 2.90427 10.3408 2.92686 10.707 3.29298L15.707 8.29298L15.7753 8.36915C16.0957 8.76192 16.0731 9.34092 15.707 9.70704C15.3408 10.0732 14.7618 10.0958 14.3691 9.7754L14.2929 9.70704L10.9999 6.41407V16C10.9999 16.5523 10.5522 17 9.99992 17C9.44764 17 8.99992 16.5523 8.99992 16Z">
              </path>
            </svg>
          </button>
        </div>
        <p class="md:text-sm text-gray-500 text-center my-2 !text-[13px]">
          <strong class="text-[13px]">Disclaimer</strong>: Messages sent to Gemini API, not stored by us; *limited data
          store on local storage*.
        </p>
      </div>
    </div>
  </main>
</template>
