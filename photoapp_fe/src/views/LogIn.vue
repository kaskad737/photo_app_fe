<template>
    <div class="container">
        <div class="column">
            <div class="column is-4 is-offset-4">
                <h1 class="title">Log in</h1>

                <form @submit.prevent="submitForm">
                    <div class="field">
                        <label>Email</label>
                        <div class="control">
                            <input type="email" name="email" v-model="email" class="input">
                        </div>
                    </div>

                    <div class="field">
                        <label>Password</label>
                        <div class="control">
                            <input type="password" name="password" v-model="password" class="input">
                        </div>
                    </div>
                    
                    <div class="field">
                        <div class="control">
                            <button class="button is-success sub-btn">Log in</button>
                        </div>
                    </div>

                </form>
            </div>
        </div>
    </div>
</template>

<script>
    import axios from 'axios';

    export default {
        name: 'LogIn',
        data() {
            return {
                username: '',
                password: ''
            }
        },
        methods: {
            submitForm(e) {
                axios.defaults.headers.common['Authorization'] = ''
                localStorage.removeItem('access')

                const formData = {
                    username: this.email,
                    password: this.password
                }

                axios
                    .post('auth/token/', formData)
                    .then(response => {
                        console.log(response)

                        const access = response.data.access
                        const refresh = response.data.refresh

                        this.$store.commit('setAccess', access)
                        this.$store.commit('setRefresh', refresh)

                        axios.defaults.headers.common['Authorization'] = "Bearer " + access

                        localStorage.setItem("access", access)
                        localStorage.setItem("refresh", refresh)

                        return axios.get('auth/users/');
                    })
                    .then((response) => {
                        const filteredUserId = response.data.results.find(user => user.username === formData.username).pk;
                        return axios.get(`auth/users/${filteredUserId}`);
                    })
                    .then((response) => {
                        const user = response.data;

                        this.$store.commit('setUser', {
                            userId: user.id,
                            username: user.username,
                            userFirstName: user.first_name,
                            userRole: user.role
                        });

                        localStorage.setItem('userId', user.id);
                        localStorage.setItem('username', user.username);
                        localStorage.setItem('userFirstName', user.first_name);
                        localStorage.setItem('userRole', user.role);

                        this.$router.push("/")
                    })
                    .catch(error => {
                        console.log(error)
                    })
            }
        }
    }
</script>

<style scoped>
    .sub-btn {
        background-color: blueviolet;
        color: white;
        font-weight: bold;
        transition: background-color 0.3s ease;
    }

    .sub-btn:hover {
        background-color: #185ea0;
    }
</style>