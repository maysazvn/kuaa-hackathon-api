<script setup>

import { ref, onMounted } from 'vue'
import ButtonChild from '@/components/ButtonChild.vue'
// import { router } from '@/router'
import { userReal } from './login/UserReal'
import { emailReal } from './login/EmailReal'
import { senhaReal } from './login/SenhaReal'
import { loginOut } from './login/Loginout'
import { users } from '../user/Users'

// lets ////////

let cadastro = ref(true)
let login = ref(false)
let emailFalso = ref('')
let userFalso = ref('')
let senhaFalsa = ref('')
let maiorId = Math.max(...users.map((item) => item.id))
const desc = ref(localStorage.getItem('desc') || '')
const mostrarSala = ref(localStorage.getItem('mostrarSala?') || 'sim')

onMounted(() => {
  const emailSalvo = localStorage.getItem('user_email')
  const userSalvo = localStorage.getItem('user_nome')
  const senhaSalva = localStorage.getItem('user_senha')
  const statusLogin = localStorage.getItem('login_status')

  if (emailSalvo && userSalvo && senhaSalva) {
    emailReal.value = emailSalvo
    userReal.value = userSalvo
    senhaReal.value = senhaSalva
  }

  if (statusLogin === 'ativo') {
    loginOut.value = 'ativo'
    cadastro.value = false
    login.value = true
  }
})

// Functions /////

function selecionarLogin() {
  if (cadastro.value == true) {
    cadastro.value = false
    login.value = true
  }
}
function selecionarCadastro() {
  if (login.value == true) {
    login.value = false
    cadastro.value = true
  }
}

 async function enviar(email, senha, user) {
  if (email.trim() !== '' && senha.trim() !== '' && user.trim() !== '') {
    if(emailFalso.value.includes(`@`) && emailFalso.value.includes('.com' || '.br' || '.ifc' || '.edu')) {

    
    if (senhaReal.value.length == 0 && emailReal.value.length == 0 && userReal.value.length == 0) {
      emailReal.value = email
      userReal.value = user
      senhaReal.value = senha

      novoLogin.value.nomeUser = user
        novoLogin.value.email = email
        novoLogin.value.senha = senha
      await salvarNaApi()

      localStorage.setItem('user_email', email)
      localStorage.setItem('user_nome', user)
      localStorage.setItem('user_senha', senha)

      emailFalso.value = ''
      senhaFalsa.value = ''
      userFalso.value = ''

       const novoUsuario = {
  id: maiorId + 1,
  nome: userReal.value,
  pfp: localStorage.getItem('urlFoto') || '/pfpPlaceholder.png',
  banner: localStorage.getItem('urlBanner') || '/bannerPlaceholder.png',
  desc: desc.value,
  mostrarSala: mostrarSala.value,
  salas: [],
      
}
    

      users.push(novoUsuario);
      localStorage.setItem('usuarios_cadastrados', JSON.stringify(users.filter(u => u.id > 30)));

      alert('Cadastro realizado com sucesso! Vá para a aba Login.')
    } else {
      alert('Você já possui cadastro! Faça login!')
    }
  }else{
    alert('Coloque um email válido!')
  }
  } else {
    alert('Preencha todos os campos!')
  }
}

function logar(email, senha) {
  if (email === emailReal.value && senha === senhaReal.value) {
    loginOut.value = 'ativo'

    localStorage.setItem('login_status', 'ativo')

    alert('Você logou com sucesso!')
    senhaFalsa.value = ''
    emailFalso.value = ''
  } else {
    alert('Erro! Senha ou/e usuário inválido!')
  }
}


//////////////////////////////////// FUNCTIONS PARA A API ////////////////////////////

const API_URL = 'http://localhost:3000';

const novoLogin = ref({
  nomeUser: '',
  email: '',
  id: '',
});

async function salvarNaApi() {
  try {
    await fetch(`${API_URL}/login`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(novoLogin.value),
    })
  } catch (error) {
    console.error('Erro ao salvar na API:', error)
  }
}




</script>

<template>
  <div class="container">
    <!-- essa ^ div é a geral que contem TUDO -->
    <div class="bemVindo">
      <!-- essa ^ tem o texto de bem vindo e a imagem do kuaa -->
      <img src="../../../public/kuaa.png" alt="Kuaa Logo" />
      <p v-if="cadastro == true">
        <span>Bem-vindo ao KUAA</span>
        O KUAA veio para ajudar você, estudante, a ter um desempenho melhor em seus estudos, provas
        e até mesmo fazer amigos novos!
      </p>
      <p v-if="login == true">
        <span>Bem-vindo de volta ao KUAA</span>
        O KUAA veio para ajudar você, estudante, a ter um desempenho melhor em seus estudos, provas
        e até mesmo fazer amigos novos!
      </p>
    </div>
    <div class="direita">
      <div class="escolha">
        <!-- essa ^ tem os botoes para alterar o login e cadastro -->
        <ButtonChild
          @clique="selecionarCadastro()"
          :class="{ inativo: !cadastro }"
          class="botaoCadastro"
          >Cadastro</ButtonChild
        >
        <ButtonChild @clique="selecionarLogin()" :class="{ inativo: !login }" class="botaoLogin"
          >Login</ButtonChild
        >
      </div>

      <div class="cadastro" v-show="cadastro == true">
        <!-- essa ^ é a div que aparece quando você esta fazendo cadastro -->
        <label for="email"></label>
        <input type="email" name="email" placeholder="E-mail" required v-model="emailFalso" pattern=".*@.*"/>
        <input type="text" required v-model="userFalso" placeholder="Nome de usuário" />
        <input type="password" required v-model="senhaFalsa" placeholder="Senha" />
        <ButtonChild @clique="enviar(emailFalso, senhaFalsa, userFalso)"  class="botaoLog"
          >Cadastrar</ButtonChild
        >
      </div>
      <div class="login" v-show="login == true">
        <!-- essa é a div ^ que aparece para fazer login -->
        <input type="email" name="email" placeholder="E-mail" required v-model="emailFalso" />
        <input type="password" required v-model="senhaFalsa" placeholder="Senha" />
        <ButtonChild @clique="logar(emailFalso, senhaFalsa)" class="botaoLog"> Logar </ButtonChild>
      </div>
    </div>
  </div>
</template>

<style scoped>
div.container {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 10px;
  padding-top: 40px;
  gap: 150px;
}

.bemVindo {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.bemVindo img {
  max-width: 380px;
  height: auto;
}

.bemVindo p {
  color: #c0c0c0;
  font-size: 1.2rem;
  line-height: 1.5;
  max-width: 380px;
}

.bemVindo p span {
  display: block;
  font-weight: bold;
  font-size: 1.5rem;
}

.direita {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.escolha {
  display: flex;
  gap: 20px;
  margin-bottom: 24px;
  justify-content: center;
}

.escolha .botaoCadastro,
.escolha .botaoLogin {
  font-family: 'Prompt', sans-serif;
  font-weight: bold;
  font-size: 2.5rem;
  color: #f8d76b;
  transition: 0.5s;
}

.escolha .inativo {
  color: #f8d76b7e;
}

.escolha .inativo:hover {
  transform: scale(0.95);
}

.cadastro,
.login {
  display: flex;
  flex-direction: column;
  gap: 10px;
  width: 20vw;
}

input {
  background-color: #2a2a2a;
  border: 1px solid #e0d8c3;
  border-radius: 10px;
  padding: 12px 20px;
  color: #e0d8c3;
  font-size: 1rem;
  outline: none;
}

.botaoLog {
  background-color: #f8d76b;
  color: #1e1e1e;
  font-weight: bold;
  font-size: 1rem;
  padding: 12px 20px;
  border-radius: 25px;
}

.botaoLog:hover {
  transition: 0.2s;
  background-color: #f8d76ba4;
  transform: scale(0.98);
}

@media (max-width: 768px) {
  .container {
    display: flex;
    flex-direction: column;
    gap: 5px !important;
  }

  img {
    width: 80%;
    margin-bottom: 25px;
  }

  p {
    font-size: 0.9rem;
  }

  input {
    width: 100%;
  }

  .cadastro{
    width: 100%;
  }

  .login{
    width: 100%;
  }

  .direita {
    padding: 0;
  }
}
</style>
