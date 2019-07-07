<template>
    <Page class="page" @loaded="onPageLoaded">
        <ActionBar title="To Do Lists" flat="true" />
        <TabView height="100%" :selectedIndex="activeTabIndex"
            @selectedIndexChange="onTabChange" tabBackgroundColor="blue"
            tabTextColor="white" selectedTabTextColor="cyan">
            <TabViewItem title="To Do" textTransform="uppercase">
                <GridLayout rows="auto, auto, *">
                    <Label text="My Tasks" row="0" class="tabTitle" />
                    <TextField ref="taskInput" v-model="textFieldValue" hint="Enter new task..."
                        row="1" @returnPress="onReturnPress" returnKeyType="done" />
                    <ListView class="list-group" for="todo in todos" @itemTap="onItemTap"
                        row="2">
                        <v-template>
                            <GridLayout columns="auto, *" class="list-entry">
                                <Label text=" " col="0" v-on:tap="onTodoCircleTap(todo)"
                                    class="circle" />
                                <Label :text="todo.name" textWrap="true" col="1"
                                    class="todoItems" v-on:tap="onTodoItemTap(todo)" />
                            </GridLayout>
                        </v-template>
                    </ListView>
                </GridLayout>
            </TabViewItem>
            <TabViewItem title="Completed Tasks" textTransform="uppercase">
                <GridLayout rows="auto, auto, *">
                    <Label text="My Completed Tasks" row="0" class="tabTitle" />
                    <ListView class="list-group" for="todo in completed"
                        @itemTap="onItemTap" row="2">
                        <v-template>
                            <GridLayout columns="auto, *" class="list-entry">
                                <Label text="✔" col="0" v-on:tap="onCompleteCircleTap(todo)"
                                    class="circle" />
                                <Label :text="todo.name" textWrap="true" col="1"
                                    class="todoItems" v-on:tap="onCompleteItemTap(todo)" />
                            </GridLayout>
                        </v-template>
                    </ListView>
                </GridLayout>
            </TabViewItem>
        </TabView>
    </Page>
</template>

<script>
    const platform = require("tns-core-modules/platform");
    const frame = require("tns-core-modules/ui/frame");
    const LS = require("nativescript-localstorage");
    export default {
        methods: {
            onTodoItemTap: function(item) {
                const index = this.todos.indexOf(item);
                action("Action on task:", "Cancel", [
                    "Mark complete",
                    "Delete forever"
                ]).then(result => {
                    console.log(result);
                    switch (result) {
                        case "Mark complete":
                            this.completed.unshift(item);
                            this.todos.splice(index, 1);
                            this.activeTabIndex = 1;
                            break;
                        case "Delete forever":
                            this.todos.splice(index, 1);
                            break;
                        case "Cancel" || undefined:
                            break;
                    }
                    LS.setItem('todos', JSON.stringify(this.todos));
                    LS.setItem('completed', JSON.stringify(this.completed));
                });
            },
            onTodoCircleTap: function(item) {
                const index = this.todos.indexOf(item);
                this.completed.unshift(item);
                this.todos.splice(index, 1);
                this.activeTabIndex = 1;
                LS.setItem('todos', JSON.stringify(this.todos));
                LS.setItem('completed', JSON.stringify(this.completed));
            },
            onCompleteItemTap: function(item) {
                const index = this.completed.indexOf(item);
                action("Action on task:", "Cancel", [
                    "Mark incomplete",
                    "Delete forever"
                ]).then(result => {
                    console.log(result);
                    switch (result) {
                        case "Mark incomplete":
                            this.todos.unshift(item);
                            this.completed.splice(index, 1);
                            this.activeTabIndex = 0;
                            break;
                        case "Delete forever":
                            this.completed.splice(index, 1);
                            break;
                        case "Cancel" || undefined:
                            break;
                    }
                    LS.setItem('todos', JSON.stringify(this.todos));
                    LS.setItem('completed', JSON.stringify(this.completed));
                });
            },
            onCompleteCircleTap: function(item) {
                const index = this.completed.indexOf(item);
                this.todos.unshift(item);
                this.completed.splice(index, 1);
                this.activeTabIndex = 0;
                LS.setItem('todos', JSON.stringify(this.todos));
                LS.setItem('completed', JSON.stringify(this.completed));
            },
            onReturnPress: function() {
                if (this.textFieldValue.trim() == "") {
                    this.$refs.taskInput.nativeView.focus();
                    return;
                }
                this.todos.unshift({
                    name: this.textFieldValue.trim()
                });
                this.textFieldValue = "";
                LS.setItem('todos', JSON.stringify(this.todos));
            },
            onTabChange: function(tab) {
                this.activeTabIndex = tab.value;
            },
            onPageLoaded: function() {
                console.log('Page loading');
                if (platform.isIOS) {
                    const navBar = frame.topmost().ios.controller.navigationBar;
                    navBar.barStyle = UIBarStyle.UIBarStyleBlack;
                    IQKeyboardManager.sharedManager().enableAutoToolbar = faAppSe;
                }
                console.log('Loading values');
                this.todos = JSON.parse(LS.getItem('todos') || "[]");
                this.completed = JSON.parse(LS.getItem('completed') || "[]");
                /*if(this.todos == null) {
                    this.todos = [];
                } else {
                    this.todos = JSON.parse(this.todos);
                }
                if(this.completed == null) {
                    this.completed = [];
                } else {
                    this.completed = JSON.parse(this.completed);
                }*/
                console.log('Values loaded');
                setInterval(() => {
                    console.log(this.todos);
                    console.log(this.completed);
                    console.log("\n");
                })
            }
        },

        data() {
            return {
                todos: [],
                completed: [],
                textFieldValue: "",
                activeTabIndex: 0
            };
        }
    };
</script>

<style scoped>
    ActionBar {
        background-color: blue;
        color: white;
    }

    TabView {
        background-color: #00CFFF;
    }

    TextField {
        width: 100%;
        font-size: 20px;
        color: blue;
        placeholder-color: white;
    }

    .todoItems {
        font-size: 20px;
        color: blue;
    }

    .tabTitle {
        font-size: 24px;
        background-color: indigo;
        color: white;
        font-weight: bold;
        padding: 5 15 5 15;
        margin: 0;
    }

    .list-entry {
        padding: 3 15 3;
    }

    .circle {
        width: 20;
        height: 20;
        padding: 0;
        font-size: 13px;
        text-align: center;
        vertical-align: middle;
        color: indigo;
        border-color: indigo;
        border-width: 5px;
        border-radius: 50;
    }

    .list-entry .circle {
        margin: 0 5 0 0;
    }
</style>