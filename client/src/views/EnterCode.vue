<template>
    <div>
        <nav-bar></nav-bar>
        <b-container fluid class="center">
            <b-card class="w-75 login-card">
                <p>{{ $socket.connected ? 'Connected' : 'Disconnected' }}</p>
                <b-form @submit="onSubmit" class="m-4 text-center">
                    <h1 v-if="countDown === 'over'">Time is over</h1>
                    <h1 v-else>Time left:  {{countDown}}</h1>
                    <h1>Enter code: </h1>
                    <input v-model="enteredCode" class="code-box my-5" required> <br>
                    <b-button type="submit" class="generate-btn">Submit</b-button>
                    <p>{{message}}</p>
                </b-form>
            </b-card>
        </b-container>
        
    </div>
</template>
<script>
import NavBar from '../components/NavBar.vue'
import {checkToken} from '../services/AuthService'
import {updateAttendanceList} from '../services/ClassService'

export default {
    components: {
        NavBar
    },
    data(){
        return {
            countDown : "01:00",
            enteredCode: "",
            message: "",
            generatedCode: "",
            user: {},
            generatedCodeClassId: ""
          
        }
    },
    sockets: {
        check(code){
            this.generatedCode = code
        },
        class(codeClass){
            console.log(codeClass);
            this.generatedCodeClassId = codeClass._id
        }
    },
    methods: {
        onSubmit(event){
            event.preventDefault()
            if((this.generatedCode === this.enteredCode) && (this.countDown !== 'over') && 
            (this.generatedCodeClassId === this.$route.params.classId)){
                updateAttendanceList(this.$route.params.classId, {studentId: this.user._id, check: true}).then(() => {
                    const today = new Date();
                    const time = today.getHours() + ":" + today.getMinutes()
                    this.$router.push(`/${this.$route.params.classId}/attendance/confirmed/${time}`)
                })
            }else{
                updateAttendanceList(this.$route.params.classId, {studentId: this.user._id, check: false}).then(() => {
                    this.$router.push(`/${this.$route.params.classId}/attendance/denied/`)
                })
                
            }
        },
        
        formatTime(seconds){
            let minutes = Math.floor(seconds / 60)
            let sec = seconds - minutes * 60;
            return minutes + ':' + sec
        }
        
    },
    created(){
        if(!checkToken(['student'])) {
            this.$router.push('/login');
        }else{
            this.user = JSON.parse(localStorage.getItem('user'))
            this.$socket.$subscribe('countdown-student', (data) => {
                this.countDown = data
            })
        }
        
        
    },  
}
</script>
<style scoped>
.center{
    display: flex;
    justify-content: center;
}
.login-card{
    border-color: #161966;
    margin: 0;
    position: absolute;
    top: 50%;
    left: 50%;
    -ms-transform: translate(-50%, -50%);
    transform: translate(-50%, -50%);
    
}
.navy{
    color: #161966;
}
.generate-btn{
    width: 25%;
    margin: 0 auto;
    height: 50px;
    background-color: #161966;
}
.code-box{
    height: 60px;
    width: 50%;
    border: solid #161966 1px;
    font-size: 25px;
    color: #E51818;
    font-weight: bold;
    text-align: center;
}
</style>
