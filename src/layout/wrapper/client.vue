<template>
    <div class="wrapper">
        <MenuClient />
        <router-view />
        <BotClient />
    </div>

    <!-- CHAT WINDOW -->
    <div class="chatwindow" v-if="isOpen">
        <!-- HEADER -->
        <div class="header">
            <div class="title">
                <span class="dot"></span>
                Hỗ Trợ Trực Tuyến
            </div>
            <button class="close" @click="isOpen = false">✖</button>
        </div>

        <!-- CHAT BOX -->
        <div class="chat" ref="chatBox">
            <div v-for="(msg, index) in messages" :key="index" :class="['message', msg.role]">
                <p>{{ msg.text }}</p>
            </div>
        </div>

        <!-- INPUT AREA -->
        <div class="inputarea">
            <input v-model="input" @keyup.enter="sendMessage" type="text" placeholder="Nhập tin nhắn..."
                :disabled="loading" />
            <button class="send" @click="sendMessage" :disabled="loading || !input.trim()">
                <span v-if="loading">...</span>
                <span v-else>➤</span>
            </button>
        </div>
    </div>

    <!-- BUTTON MỞ CHAT -->
    <button class="open-chat" v-if="!isOpen" @click="isOpen = true">
        💬
    </button>
</template>

<script>
import axios from "axios";
import MenuClient from "../components/MenuClient.vue";
import BotClient from "../components/BotClient.vue";

export default {
    name: "AppLayout",
    components: { MenuClient, BotClient },

    data() {
        return {
            isOpen: true,
            input: "",
            loading: false,
            messages: [
                { role: "bot", text: "Xin chào! MP Cinema có thể giúp gì cho bạn?" }
            ]
        };
    },

    methods: {
        async sendMessage() {
            const trimmedInput = this.input.trim();
            if (!trimmedInput || this.loading) return;

            const userMessage = trimmedInput;

            // Hiển thị tin nhắn người dùng
            this.messages.push({ role: "user", text: userMessage });
            this.input = "";
            this.loading = true;

            // Tạo tin nhắn chờ cho Bot
            const loadingIndex = this.messages.push({ role: "bot", text: "Đang suy nghĩ..." }) - 1;
            this.scrollToBottom();

            try {
                // Đảm bảo URL này khớp với route:post('/client/chat') hoặc route tương ứng của bạn
                const res = await axios.post("http://127.0.0.1:8000/api/client/chat", {
                    message: userMessage
                });

                // Cập nhật câu trả lời (Dùng res.data.reply khớp với Backend ở trên)
                this.messages[loadingIndex].text = res.data?.reply || "AI không phản hồi.";

            } catch (error) {
                console.error("Lỗi:", error);
                this.messages[loadingIndex].text = error.response?.data?.reply || "Lỗi kết nối server rồi 😢";
            } finally {
                this.loading = false;
                this.scrollToBottom();
            }
        },

        scrollToBottom() {
            this.$nextTick(() => {
                const chatContainer = this.$refs.chatBox;
                if (chatContainer) {
                    chatContainer.scrollTop = chatContainer.scrollHeight;
                }
            });
        }
    }
};
</script>

<style scoped>
/* ================= WRAPPER ================= */
.chatwindow {
    position: fixed;
    bottom: 90px;
    right: 25px;
    width: 370px;
    height: 550px;
    background: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(12px);
    border-radius: 24px;
    box-shadow:
        0 10px 30px rgba(0, 0, 0, 0.15),
        0 0 0 1px rgba(255, 255, 255, 0.3);
    display: flex;
    flex-direction: column;
    overflow: hidden;
    z-index: 9999;
    animation: popup 0.3s ease;
}

/* Animation mở chat */
@keyframes popup {
    from {
        transform: translateY(30px) scale(0.9);
        opacity: 0;
    }

    to {
        transform: translateY(0) scale(1);
        opacity: 1;
    }
}

/* ================= HEADER ================= */
.header {
    background: linear-gradient(135deg, #4e73df, #224abe);
    color: white;
    padding: 16px 18px;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.title {
    display: flex;
    align-items: center;
    font-size: 16px;
    font-weight: 600;
    gap: 10px;
}

.dot {
    width: 10px;
    height: 10px;
    background: #1cc88a;
    border-radius: 50%;
    box-shadow: 0 0 10px #1cc88a;
}

.close {
    background: rgba(255, 255, 255, 0.15);
    border: none;
    color: white;
    width: 34px;
    height: 34px;
    border-radius: 50%;
    cursor: pointer;
    transition: 0.2s;
    font-size: 14px;
}

.close:hover {
    background: rgba(255, 255, 255, 0.3);
    transform: rotate(90deg);
}

/* ================= CHAT BODY ================= */
.chat {
    flex: 1;
    padding: 18px;
    overflow-y: auto;
    background:
        linear-gradient(to bottom right, #f8f9fc, #eef2ff);
    display: flex;
    flex-direction: column;
    gap: 10px;
}

/* Thanh cuộn đẹp */
.chat::-webkit-scrollbar {
    width: 6px;
}

.chat::-webkit-scrollbar-thumb {
    background: rgba(0, 0, 0, 0.15);
    border-radius: 10px;
}

/* ================= MESSAGE ================= */
.message {
    display: flex;
    animation: fadeIn 0.25s ease;
}

@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateY(10px);
    }

    to {
        opacity: 1;
        transform: translateY(0);
    }
}

.message p {
    padding: 12px 16px;
    border-radius: 18px;
    max-width: 80%;
    font-size: 14px;
    line-height: 1.5;
    margin: 0;
    word-wrap: break-word;
    box-shadow: 0 3px 8px rgba(0, 0, 0, 0.05);
}

/* BOT */
.message.bot {
    justify-content: flex-start;
}

.message.bot p {
    background: white;
    color: #333;
    border-bottom-left-radius: 5px;
}

/* USER */
.message.user {
    justify-content: flex-end;
}

.message.user p {
    background: linear-gradient(135deg, #4e73df, #224abe);
    color: white;
    border-bottom-right-radius: 5px;
}

/* ================= INPUT ================= */
.inputarea {
    display: flex;
    align-items: center;
    padding: 12px;
    background: white;
    border-top: 1px solid #edf0f7;
}

.inputarea input {
    flex: 1;
    border: 1px solid #dfe3f0;
    border-radius: 30px;
    padding: 12px 16px;
    outline: none;
    font-size: 14px;
    transition: 0.2s;
    background: #f8f9fc;
}

.inputarea input:focus {
    border-color: #4e73df;
    background: white;
    box-shadow: 0 0 0 3px rgba(78, 115, 223, 0.15);
}

/* BUTTON SEND */
.send {
    background: linear-gradient(135deg, #4e73df, #224abe);
    border: none;
    color: white;
    width: 46px;
    height: 46px;
    margin-left: 10px;
    border-radius: 50%;
    cursor: pointer;
    font-size: 18px;
    transition: 0.25s;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: 0 5px 15px rgba(78, 115, 223, 0.3);
}

.send:hover {
    transform: scale(1.08);
}

.send:disabled {
    opacity: 0.6;
    cursor: not-allowed;
}

/* ================= OPEN BUTTON ================= */
.open-chat {
    position: fixed;
    bottom: 25px;
    right: 25px;
    background: linear-gradient(135deg, #4e73df, #224abe);
    color: white;
    border: none;
    width: 65px;
    height: 65px;
    border-radius: 50%;
    font-size: 28px;
    cursor: pointer;
    z-index: 9998;
    box-shadow: 0 8px 20px rgba(78, 115, 223, 0.4);
    transition: 0.3s;
    animation: float 2s infinite ease-in-out;
}

.open-chat:hover {
    transform: scale(1.1);
}

/* Floating animation */
@keyframes float {
    0% {
        transform: translateY(0px);
    }

    50% {
        transform: translateY(-5px);
    }

    100% {
        transform: translateY(0px);
    }
}

/* ================= RESPONSIVE ================= */
@media (max-width: 500px) {
    .chatwindow {
        width: 95%;
        height: 90vh;
        right: 2.5%;
        bottom: 10px;
        border-radius: 20px;
    }

    .open-chat {
        right: 15px;
        bottom: 15px;
    }
}
</style>