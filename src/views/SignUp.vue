<template>
    <v-app>
        <div class="login-box">
            <v-card class="login-form">
                <v-card-title class="login-title">SignUp</v-card-title>
                <v-card-subtitle>ユーザー情報を入力してください</v-card-subtitle>
                <v-btn text color="light-blue" to="login">ログインはこちら</v-btn>
                <v-form
                    ref="form"
                    v-model="valid"
                    lazy-validation
                >

                <v-text-field
                    v-model="name"
                    :rules="nameRules"
                    label="UserName"
                    required
                ></v-text-field>

                <v-text-field
                    v-model="email"
                    :rules="emailRules"
                    label="E-mail"
                    required
                ></v-text-field>

                <v-text-field
                    v-model="password"
                    type="password"
                    label="Password"
                    required
                ></v-text-field>

                <v-btn
                color="success"
                class="login-btn"
                @click="submit"
                :disabled="isValid">
                    SIGN UP
                </v-btn>
                <v-btn>
                    Clear
                </v-btn>
                <v-alert
                    dense
                    outlined
                    type="error"
                    class="error-message"
                    v-if="errorMessage"
                >
                {{errorMessage}}
                </v-alert>
                </v-form>
            </v-card>
        </div>
    </v-app>
</template>

<script>
    import firebase from "@/firebase/firebase"
    export default {
    data: () => ({
        valid: true,
        name: '',
        nameRules: [
        v => !!v || '名前を入力してください',
        v => (v && v.length <= 10) || '名前は10文字以内で入力してください',
        ],
        email: '',
        emailRules: [
            v => !!v || 'E-mailを入力してください',
        v => /.+@.+\..+/.test(v) || 'E-mailが正しくありません',
        ],
        password: '',
        errorMessage: ''
    }),
    computed: {
        isValid(){
            console.log(this.valid);
            return !this.valid
        }
    },
    methods: {
        validate () {
        this.$refs.form.validate()
        },
        reset () {
        this.$refs.form.reset()
        },
        resetValidation () {
        this.$refs.form.resetValidation()
        },
        submit(){
            console.log("submit call");
            firebase.auth().createUserWithEmailAndPassword(this.email,this.password)
            .then(async(result)=>{
                await result.user.updateProfile(
                    {displayName:this.name}
                );
                console.log("update user",result.user);

                localStorage.message= "新規作成に成功しました"
                // loginにリダイレクト
                this.$router.push('/login')
            })
            .catch((error)=>{
                console.log("fail",error)
                // エラーメッセージを表示
                this.errorMessage="ユーザーの新規作成に失敗しました";
            })
        }
    },
    }
</script>

<style scoped>
.login-title{
    display: inline-block;
}
.login-box{
    width: 75%;
    margin: 0 auto;
    padding: 30px;
}
.login-form{
    margin: 50px;
    padding: 30px;
}
.login-btn{
    margin-right: 20px;
}
.error-message{
    margin-top: 20px;
}
</style>