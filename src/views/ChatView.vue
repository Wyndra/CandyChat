<template>
    <n-layout id="chatview" content-style="padding: 6%; padding-left: 15%; padding-right: 15%" :native-scrollbar="false"
        ref="contentRef" style="bottom: 64px;margin-top: 50px;">
        <div v-for="item in MessageList" :key="item.id" style="margin-bottom: 2%;">
            <n-card :title="item.role" width hoverable size="small" style="padding: 1%;">
                <n-card-content>
                    <div v-html="renderMarkdown(item.content)"></div>
                </n-card-content>
                <template #footer v-if='item.role != "你 🤔"'>
                    <div class="copy_button" @click="copyToClipboard(item.content)">
                        <n-icon size="15" class="copy_icon_">
                            <svg t="1718343191961" class="icon" viewBox="0 0 1024 1024" version="1.1"
                                xmlns="http://www.w3.org/2000/svg" p-id="6898" width="200" height="200">
                                <path
                                    d="M407.424 53.312H403.2c-68.352 0-122.56 0-165.376 5.12-43.968 5.12-80.192 16.064-110.464 40.96-10.24 8.32-19.648 17.792-28.096 28.032-24.832 30.272-35.712 66.432-40.96 110.464C53.312 280.768 53.312 334.912 53.312 403.2v4.16c0 68.352 0 122.496 5.12 165.376 5.12 43.968 16.064 80.192 40.96 110.464 8.32 10.24 17.792 19.648 28.032 28.032 28.672 23.552 62.72 34.56 103.68 40.128 40.064 5.376 90.112 5.824 152.448 5.888H384a32.128 32.128 0 0 0 32-32.832V597.312a32 32 0 0 0-64 0v95.936c-47.36-0.32-83.392-1.344-112.384-5.248-34.496-4.672-55.488-12.928-71.616-26.176-7.04-5.76-13.44-12.16-19.2-19.2-13.888-16.96-22.4-39.424-26.88-77.376-4.48-38.656-4.608-88.96-4.608-159.936 0-70.912 0.064-121.28 4.672-159.936 4.48-37.952 12.928-60.416 26.88-77.376 5.76-7.04 12.16-13.44 19.2-19.2 16.896-13.888 39.36-22.4 77.312-26.88 38.656-4.48 89.024-4.608 160-4.608 70.848 0 121.28 0.064 159.872 4.672 37.952 4.48 60.416 12.928 77.44 26.88 6.976 5.76 13.44 12.16 19.2 19.2 13.184 16 21.44 37.12 26.112 71.552 3.84 28.992 4.928 65.024 5.248 112.384H597.312a32 32 0 0 0 0 64h128.064a32 32 0 0 0 32-32c-0.128-62.592-0.512-112.704-5.952-152.96-5.504-40.96-16.576-74.944-40.128-103.616a202.88 202.88 0 0 0-28.032-28.096c-30.272-24.832-66.496-35.712-110.464-40.96C529.92 53.312 475.776 53.312 407.424 53.312z"
                                    fill="#333333" p-id="6899"></path>
                                <path
                                    d="M493.12 415.744a32 32 0 1 0-8.32-63.488c-34.816 4.544-64.512 13.696-88.896 33.728-24.704 20.352-37.12 46.272-43.264 77.056a32 32 0 1 0 62.72 12.608c4.096-20.48 10.752-31.616 21.184-40.192 11.328-9.344 27.52-15.936 56.576-19.712zM837.824 352.256a32 32 0 1 0-8.32 63.488c29.056 3.84 45.248 10.368 56.576 19.712 10.432 8.576 17.088 19.712 21.184 40.192a32 32 0 1 0 62.72-12.608c-6.144-30.72-18.56-56.704-43.264-77.056-24.384-20.032-54.08-29.184-88.896-33.728zM970.624 597.312a32 32 0 0 0-64 0v128a32 32 0 0 0 64 0v-128zM415.36 847.04a32 32 0 0 0-62.72 12.608c6.144 30.72 18.56 56.64 43.264 76.992 24.384 20.096 54.08 29.184 88.96 33.728a32 32 0 0 0 8.32-63.424c-29.056-3.84-45.312-10.368-56.64-19.712-10.432-8.576-17.088-19.712-21.12-40.192zM970.048 859.648a32 32 0 1 0-62.72-12.608c-4.16 20.48-10.816 31.616-21.248 40.192-11.328 9.344-27.52 15.872-56.576 19.712a32 32 0 0 0 8.32 63.424c34.816-4.48 64.512-13.632 88.96-33.728 24.704-20.352 37.12-46.272 43.264-76.992zM597.312 906.688a32 32 0 0 0 0 64h128a32 32 0 0 0 0-64h-128z"
                                    fill="#333333" p-id="6900"></path>
                            </svg>
                        </n-icon>
                    </div>
                </template>
            </n-card>
        </div>
    </n-layout>
</template>

<script setup>
import { useRoute } from 'vue-router';
import { useMessage } from 'naive-ui';
import { ref, onMounted, watch, nextTick } from 'vue';
import MarkdownIt from 'markdown-it';
import hljs from 'highlight.js';
import 'highlight.js/styles/github-dark.css';
import markdownItHighlightjs from 'markdown-it-highlightjs';

// Configure MarkdownIt with highlight.js
const markdown = new MarkdownIt().use(markdownItHighlightjs, { hljs });

const contentRef = ref(null);
const route = useRoute();
const MessageList = ref([]);
const message = useMessage();

onMounted(() => {
    updateData();
    window.addEventListener('updateChatView', updateData);
    nextTick(() => {
        scrollToBottom();
    });
});

watch(() => route.query.id, () => {
    MessageList.value = [];
    updateData();
    nextTick(() => {
        scrollToBottom();
    });
});

function updateData() {
    let id = route.query.id;
    for (let i = 0; i < localStorage.length; i++) {
        let key = localStorage.key(i);
        let item = JSON.parse(localStorage.getItem(key));
        console.log(item);
        if (item.id === id) {
            MessageList.value = item.messages;
        }
    }
    scrollToBottom();
}

function renderMarkdown(content) {
    return markdown.render(content);
}

function scrollToBottom() {
    nextTick(() => {
        const container = contentRef.value?.$el;
        if (container) {
            container.scrollTop = container.scrollHeight;
        }
    });
}

function copyToClipboard(content) {
    // 粘贴到剪贴板
    navigator.clipboard.writeText(content);
    // 提示
    message.success('已复制到剪贴板');
}
</script>

<style lang="scss">
.n-card__footer {
    display: flex;
    justify-content: flex-start;
    align-items: center;
    gap: 5px;
}

.copy_button {
    cursor: pointer;
}

.copy_icon_ {
    padding: 3px !important;
    border-radius: 25%;
}

.copy_icon_:hover {
    background-color: #e8e8e8;
}

body {
    font-family: Quotes, sans-serif;
}

.n-card-header {
    text-align: left;
}

.n-card__content {
    text-align: left;
}

pre code {
    background: #f5f5f5;
    padding: 10px;
    border-radius: 5px;
}
</style>
