<template>
    <div class="hello">
        <div class="container">
            <div class="row">
                <div class="col-md-8 col-md-offset-2">
                    <h1 class="page-header">
                        SwampStack
                        <img
                            :src="user.avatarUrl() ? user.avatarUrl() : '/avatar-placeholder.png'"
                            class="avatar">
                        <small>
                            <span class="sign-out">
                                (
                                <a href="#" @click.prevent="signOut">Sign Out</a>)
                            </span>
                        </small>
                    </h1>
                    <h2 class="user-info">
                        <!-- <small>
              {{ user.name() ? user.name() : 'Nameless Person'}}'s Todos
            </small>-->
                        <small class="pull-right">{{ user.username ? user.username : user.identityAddress }}</small>
                    </h2>
                    <!-- <form @submit.prevent="addTodo" :disabled="! todo">
                        <div class="input-group">
                            <input
                                v-model="todo"
                                type="text"
                                class="form-control"
                                placeholder="Write a todo..."
                                autofocus>
                            <span class="input-group-btn">
                                <button class="btn btn-default" type="submit" :disabled="! todo">Add</button>
                            </span>
                        </div>
                    </form> -->

                    <ul class="list-group">
                        <li
                            v-for="todo in todos"
                            class="list-group-item"
                            :class="{completed: todo.completed}"
                            :key="todo.id">
                            <label>
                                <input type="checkbox" v-model="todo.completed">
                                {{ todo.text }}
                            </label>
                            <a
                                @click.prevent="todos.splice(todos.indexOf(todo), 1)"
                                class="delete pull-right"
                                href="#">X</a>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from "axios";
var STORAGE_FILE = "todos.json";
const particle_access_token = "f63f36d1752b36d8a678cdc22ad98de51ef8b0d9";

export default {
    name: "dashboard",
    props: ["user"],
    data() {
        return {
            blockstack: window.blockstack,
            todos: [],
            todo: "",
            uidCount: 0
        };
    },
    watch: {
        todos: {
            handler: function(todos) {
                const blockstack = this.blockstack;

                // encryption is now enabled by default
                return blockstack.putFile(STORAGE_FILE, JSON.stringify(todos));
            },
            deep: true
        }
    },
    mounted() {
        this.fetchData();
        const url = `https://api.particle.io/v1/devices/34003e000d47373334323233/readall?access_token=${particle_access_token}`;
        axios
            .post(url, {
                args: "data"
            })
            .then(response => {
                console.log(response);
                setInterval(() => {
                    const url = `https://api.particle.io/v1/devices/34003e000d47373334323233/data?access_token=${particle_access_token}`;
                    axios.get(url).then(response => {
                        const temp = response.data.result.split(",")[0];
                        const airPressure = response.data.result.split(",")[1];
                        const lightLevel = response.data.result.split(",")[2];
                        const waterLevel = response.data.result.split(",")[3];

                        const dataItem = {
                            temp: temp,
                            airPressure: airPressure,
                            lightLevel: lightLevel,
                            waterLevel: waterLevel,
                            created: new Date().toISOString()
                        };
                        console.log(dataItem);
                        this.addItem(dataItem);
                    });
                }, 5000);
            })
            .catch(error => {
                console.log(error);
            });
    },
    methods: {
        addItem(item) {
            this.todos.unshift({
                id: this.uidCount++,
                text: JSON.stringify(item),
                completed: false
            });
        },

        fetchData() {
            const blockstack = this.blockstack;
            blockstack
                .getFile(STORAGE_FILE) // decryption is enabled by default
                .then(todosText => {
                    var todos = JSON.parse(todosText || "[]");
                    todos.forEach(function(todo, index) {
                        todo.id = index;
                    });
                    this.uidCount = todos.length;
                    this.todos = todos;
                });
        },

        signOut() {
            this.blockstack.signUserOut(window.location.href);
        }
    }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
input::placeholder {
    color: grey;
}

label {
    margin-bottom: 0;
    // width: 100%;
    cursor: pointer;
    input[type="checkbox"] {
        margin-right: 5px;
    }
}
.list-group-item {
    label {
        font-size: 10;
    }
    &.completed label {
        text-decoration: line-through;
    }

    .delete {
        display: none;
    }

    &:hover .delete {
        display: inline;
        color: grey;
        &:hover {
            text-decoration: none;
            color: red;
        }
    }
}
</style>
