<template>
    <div class="wrapper">
        <div class="get-massages-wrapper">
            <div class="single-message-wrapper" v-for="(item,id) in computedPosts" :key="id">
                {{item.author}}: {{item.message}}
                <div class="message-inf">
                    <span>email: {{item.email}}</span>
                    <span>author: {{item.author}}</span>
                    <span>Created: {{item.createdAt}}</span>
                    <span>Update: {{item.updatedAt}}</span>
                </div>
            </div>
            <div class="button-wrapper">
                <button class="arrow-back" @click="getMessagesPrev" :disabled="isPrevPageDisable"> back</button>
                <span class="count">{{count}}</span>
                <button class="arrow-next" @click="getMessagesNext" :disabled="isNextPageDisable"> next</button>
            </div>
        </div>
        <div class="message">
            <span class="email-error" v-if="errorMessage">{{errorMessage}}</span>
            <div class="information">
                <div class="email-wrapper">
                    <label for="email">Email</label>
                    <input :class="{'error': !email.length && errorMessage}" class="input email" id="email" type="email" v-model="email">
                </div>
                <div class="author-wrapper">
                    <label for="author">Author</label>
                    <input :class="{'error': !author.length && errorMessage}" class="input author" id="author" type="text" v-model="author">
                </div>
            </div>
            <input class="input text-input" :class="{'error': !text.length && errorMessage}" type="text" v-model="text"/>
            <button class="button" @mouseover="setErrorMessage" :disabled-state="isSubmitDisabled" @click="onClickSend">
                Send
            </button>
        </div>
    </div>
</template>

<script>
    import {HTTP} from '../api';

    export default {
        name: 'HelloWorld',
        data() {
            return {
                text: '',
                email: '',
                author: '',
                errors: [],
                messages: [],
                count: 1,
                urlCount: 0,
                errorMessage: null,
                prevPage: null,
                nextPage: null
            };
        },
        computed: {
            computedPosts() {
                const data = this.messages.docs || [];
                return data;
            },
            isSubmitDisabled() {
                return !this.author.length || !this.isEmailValid || !this.text.length;
            },
            isNextPageDisable() {
                return !this.nextPage
            },
            isPrevPageDisable() {
                return !this.prevPage
            },
            isEmailValid() {
                const re = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
                return re.test(String(this.email).toLowerCase());
            }
        },
        created() {
            HTTP.get('messages/list/0').then(response => {
                this.messages = response.data;
                this.messages.docs.reverse()
                this.nextPage = response.data.hasNextPage
                this.prevPage = response.data.hasPrevPage
            }).catch(e => {
                this.errors.push(e);
                console.log(this.errors);
            });
        },

        methods: {
            async onClickSend() {
                if (!this.isSubmitDisabled) {
                    await this.postMessage();
                }
            },

            async postMessage() {
                this.count = 1
                this.urlCount = 0
                if (!this.author) {
                    this.author = 'noname'
                }

                const formData = {
                    message: this.text,
                    email: this.email,
                    author: this.author,
                };

                try {
                    const {data} = await HTTP.post('messages/single', formData);
                    this.messages.docs.push(data);
                    const newArray = this.messages.docs.shift()
                    console.log(newArray)
                } catch (e) {
                    console.log(e);
                }
            },

            getMessagesNext() {
                this.count++;
                this.urlCount++;
                HTTP.get(`messages/list/${this.urlCount}`).then(response => {
                    this.messages = response.data;
                    this.nextPage = response.data.hasNextPage
                    this.prevPage = response.data.hasPrevPage
                    console.log(this.nextPage)
                    this.messages.docs.reverse()
                }).catch(e => {
                    this.errors.push(e);
                    console.log(this.errors);
                });
            },

            getMessagesPrev() {
                this.count--;
                this.urlCount--;
                HTTP.get(`messages/list/${this.urlCount}`).then(response => {
                    this.messages = response.data;
                    this.prevPage = response.data.hasPrevPage
                    this.nextPage = response.data.hasNextPage
                    this.messages.docs.reverse()
                }).catch(e => {
                    this.errors.push(e);
                    console.log(this.errors);
                });
            },

            setErrorMessage() {
                this.errorMessage = this.isSubmitDisabled ?
                    `Please fill all required fields:
            ${!this.author.length ? ' Author,' : ''}
            ${!this.email.length ? ' Email,' : ''}
            ${!this.text.length ? ' Message,' : ''}!
            ${!this.isEmailValid && this.email.length ? 'Please check if email is valid.' : ''}` : null;
            },
        }
    };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .wrapper {
        display: flex;
        flex-direction: column;
        align-items: center;
        width: 100%;
        height: fit-content;
        padding: 30px;
        background-color: gray;
    }

    .get-massages-wrapper {
        display: flex;
        flex-direction: column;
        background-color: black;
        justify-content: space-between;
        width: 50%;
        padding: 20px;
        height: 750px;
    }

    .single-message-wrapper {
        display: flex;
        flex-direction: column;
        border: 1px solid lightgray;
        justify-content: space-between;
        align-items: flex-start;
        background-color: white;
        margin-bottom: 5px;
        padding: 5px;
    }

    .message-inf {
        display: flex;
        border: 1px solid lightgray;
        justify-content: space-between;
        width: 100%;
        font-size: 10px;
    }


    .information {
        display: flex;
    }

    .message {
        display: flex;
        flex-direction: column;
        padding: 20px;
        align-items: center;
        justify-content: space-between;
        width: 50%;
        min-height: 200px;
        background-color: black;
    }

    .text-input {
        width: 80%;
        min-height: 80px;
        padding: 20px;
        outline: none;
    }

    .input.error {
        border-color: red;
    }

    .email-wrapper {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }

    .email {
        width: 200px;
        padding: 10px;
        margin-bottom: 10px;
        margin-right: 10px;
    }

    .email-error {
        color: red;
        font-size: 14px;
    }

    .author-wrapper {
        display: flex;
        flex-direction: column;
    }

    .author {
        width: 200px;
        padding: 10px;
        margin-bottom: 10px;
    }

    .button {
        width: 200px;
        height: 50px;
        cursor: pointer;
        background-color: black;
        color: white;
        margin-top: 10px;
        border-color: white;
        font-size: 20px;
        outline: none;
    }

    .button[disabled-state] {
        opacity: 0.5;
        cursor: not-allowed;
    }

    .count{
        color: white;
    }

    .input{
        outline: none;
    }

    label{
        color: white;
    }

    h3 {
        margin: 40px 0 0;
    }

    ul {
        list-style-type: none;
        padding: 0;
    }

    li {
        display: inline-block;
        margin: 0 10px;
    }

    a {
        color: #42b983;
    }
</style>
