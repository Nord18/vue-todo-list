<template>
    <div id="app">
        <v-app>
            <v-content>
                <h1 class="app__title text-xs-center font-weight-light">My todo list by Vue JS</h1>
                <v-container>
                    <v-layout justify-center>
                        <v-flex xs12 md7 lg5>
                            <v-text-field clearable :disabled="off" @input="filter = 'searching'" v-model.trim="search" color="#66BB6A" label="Search..." outline></v-text-field>
                        </v-flex>
                    </v-layout>
                    <v-layout justify-center>
                        <v-btn @click="checkAll" :disabled="off" color="white" class="app__btn">
                            <v-icon color="#66BB6A">{{icon}}</v-icon>
                        </v-btn>
                        <v-flex xs12 md5 lg4>
                            <v-text-field clearable color="secondary" type="text" ref="todoTitle" v-model.trim="todo" @keydown.enter="addTodo" label="Solo" placeholder="What needs to be done?" solo></v-text-field>
                        </v-flex>
                    </v-layout>
                    <v-layout justify-center>
                        <transition name="fade">
                            <v-flex xs12 md6 lg4 v-show="todoList.length">
                                <v-card class="app__card">
                                    <v-list>
                                        <v-list-tile v-for="item, index in filterTodo" :key="item.id">
                                            <v-list-tile-action>
                                                <v-checkbox :disabled="item.disabled" v-model="item.completed" @change="showBtn" color="success"></v-checkbox>
                                            </v-list-tile-action>
                                            <v-list-tile-content>
                                                <v-list-tile-title @click="edit(item)" :class="{'completed-text': item.completed}">{{item.title}}</v-list-tile-title>
                                                <v-text-field @keydown.enter="editDone(item)" @keydown.esc="editHide(item)" @blur="editHide(item)" v-focus class="app__field" v-show="item.editField" v-model.trim="item.title" solo></v-text-field>
                                            </v-list-tile-content>
                                            <v-list-tile-action>
                                                <v-btn @click="removeTodo(index)" color="red" small dark fab flat>
                                                    <v-icon>clear</v-icon>
                                                </v-btn>
                                            </v-list-tile-action>
                                        </v-list-tile>
                                        <v-list-tile class="app__tabs">
                                            <v-list-tile-content>
                                                <v-tabs slider-color="success">
                                                    <v-tab @click="filter = 'all'" class="success--text">All</v-tab>
                                                    <v-tab @click="filter = 'active'" class="success--text">Active</v-tab>
                                                    <v-tab @click="filter = 'completed'" class="success--text">Completed</v-tab>
                                                </v-tabs>
                                            </v-list-tile-content>
                                        </v-list-tile>
                                        <v-list-tile>
                                            <v-list-tile-title>{{completeTodo()}} items left</v-list-tile-title>
                                        </v-list-tile>
                                    </v-list>
                                </v-card>
                                <div v-show="show">
                                    <v-btn block dark @click="clearCompleted" color="#66BB6A">Clear completed</v-btn>
                                </div>
                            </v-flex>
                        </transition>
                    </v-layout>
                </v-container>
            </v-content>
        </v-app>
    </div>
</template>
<script>
export default {
    data() {
        return {
            todo: '',
            search: '',
            filter: 'all',
            todoList: [],
            show: false,
            off: true,
            icon: 'check_box_outline_blank',
            filters: {
                all: todoList => {
                    return todoList
                },
                active: todoList => {
                    return todoList.filter(el => {
                        return !el.completed
                    })
                },
                completed: todoList => {
                    return todoList.filter(el => {
                        return el.completed
                    })
                },
                searching: todoList => {
                    return todoList.filter(el => {
                        return el.title.match(this.search);
                    })
                }
            }
        }
    },
    methods: {
        addTodo() {
            if (this.$refs.todoTitle.value) {
                this.todoList.push({
                    title: this.todo,
                    id: Math.random(),
                    disabled: false,
                    editField: false,
                    completed: false
                })
                this.todo = '';
                this.show = false;
                this.icon = 'check_box_outline_blank'
            }
        },
        removeTodo(index) {
            this.$delete(this.todoList, index);
            if (this.todoList.length === 0) {
                this.icon = 'check_box_outline_blank'
            }
        },
        completeTodo() {
            return this.filters.active(this.todoList).length
        },
        clearCompleted() {
            this.todoList = this.filters.active(this.todoList);
            this.show = false;
            this.icon = 'check_box_outline_blank'
        },
        showBtn() {
            this.filters.active(this.todoList).length < this.todoList.length ? this.show = true : this.show = false;
            this.todoList.every(this.checkAllIcon) ? this.icon = 'check_box' : this.icon = 'check_box_outline_blank'
        },
        checkAllIcon(item) {
            return item.completed === true;
        },
        edit(item) {
            item.editField = true;
            item.disabled = true
        },
        editDone(item) {
            if (item.title === '') {
                return
            } else {
                item.title = item.title
            };
            item.disabled = false;
            item.editField = false
        },
        editHide(item) {
            if (item.title === '') {
                return
            } else {
                item.disabled = false;
                item.editField = false
            }
        },
        checkAll() {
            if (this.filters.active(this.todoList).length === 0) {
                this.todoList.forEach(item => {
                    item.completed = false;
                    this.show = false;
                    this.icon = 'check_box_outline_blank'
                })
            } else {
                this.todoList.forEach(item => {
                    item.completed = true;
                    this.show = true;
                    this.icon = 'check_box'
                })
            }
        }
    },
    computed: {
        filterTodo() {
            return this.filters[this.filter](this.todoList);
        }
    },
    directives: {
        focus: {
            inserted(el) {
                el.focus()
            }
        }
    },
    updated() {
        if (this.todoList.length) {
            this.off = false
        } else {
            this.off = true;
            this.search = ''
        }
    }
}

</script>
<style lang="scss" scoped>
@import '~vuetify/dist/vuetify.min.css';

$mainColor: #66BB6A;

.completed-text {
    color: $mainColor;
}

.app {
    &__title {
        font-size: 112px;
        color: $mainColor;
        opacity: .5;

        @media screen and (max-width: 1264px) {
            font-size: 56px;
        }

        @media screen and (max-width: 600px) {
            font-size: 24px;
        }
    }

    &__field {
        width: 100%;
    }

    &__btn {
        margin: 0;
        border-radius: 0;
        height: 48px;
    }

    &__card {
        margin-bottom: 20px;
    }

    &__tabs {}
}

.fade-enter {
    opacity: 0;
}

.fade-enter-active {
    transition: all 0.5s;
}

.fade-leave-active {
    opacity: 0;
    transition: all 0.5s;
}

</style>
