<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import ButtonChild from '@/components/ButtonChild.vue'

import { loginOut } from './login/Loginout'
import { users } from '../user/Users'
import { formatNumber } from 'vuetify/lib/components/VNumberInput/format.mjs'



const router = useRouter()
const API_URL = 'http://localhost:3000'

let cadastro = ref(true)
let login = ref(false)

const formulario = ref({
  email: '',
  nome: '',
  senha: ''
})
const emailReal = ref(null)
const senhaReal = ref(null)
const userReal = ref(null)
const idUser = ref(null);

// export function useUser() {
//   return {
//     idUser
//   }
// }
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

function selecionarLogin() {
  cadastro.value = false
  login.value = true
}

function selecionarCadastro() {
  login.value = false
  cadastro.value = true
}

async function cadastrarUsuario() {
  try {
    const resposta = await fetch(`${API_URL}/users`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        nome: formulario.value.nome,
        email: formulario.value.email,
        senha: formulario.value.senha,
        pfp: '/pfpPlaceholder.png',
        banner: '/bannerPlaceholder.png',
        desc: '',
        mostrarSala: 'sim',
        salas: []
      }),
    })

    if (resposta.ok) {
      localStorage.setItem('user_email', formulario.value.email)
      localStorage.setItem('user_nome', formulario.value.nome)
      localStorage.setItem('user_senha', formulario.value.senha)
      
      alert('Cadastro realizado com sucesso! Faça login.')
      emailReal.value = formulario.value.email;
      senhaReal.value = formulario.value.senha;
      userReal.value = formulario.value.user;
      formulario.value.senha = '';
      formulario.value.nome = '';
      formulario.value.email = '';
      
      selecionarLogin()
    }
  } catch (error) {
    console.error('Erro ao cadastrar:', error)
  }
}

async function logarUsuario() {
  try {
    const resposta = await fetch(`${API_URL}/users`)
    const listaUsuarios = await resposta.json()

    const usuarioEncontrado = listaUsuarios.find(
      (u) => u.email === formulario.value.email && u.senha === formulario.value.senha
    )

    if (usuarioEncontrado) {
      loginOut.value = 'ativo'
      localStorage.setItem('login_status', 'ativo')
      localStorage.setItem('user_nome', usuarioEncontrado.nome)
      localStorage.setItem('user_email', usuarioEncontrado.email)
      idUser.value = usuarioEncontrado.id
      alert('Você logou com sucesso!')
      router.push('/profile') // Redireciona para o perfil
    } else {
      alert('Erro! E-mail ou senha inválidos!')
    }
  } catch (error) {
    console.error('Erro ao fazer login:', error)
  }
}
</script>

<template>
  <div class="container">
    <div class="bemVindo">
      <img src="../../../public/kuaa.png" alt="Kuaa Logo" />
      <p v-if="cadastro">
        <span>Bem-vindo ao KUAA</span>
        O KUAA veio para ajudar você, estudante, a ter um desempenho melhor em seus estudos!
      </p>
      <p v-if="login">
        <span>Bem-vindo de volta ao KUAA</span>
        Que bom te ver de novo por aqui!
      </p>
    </div>

    <div class="direita">
      <div class="escolha">
        <ButtonChild @clique="selecionarCadastro()" :class="{ inativo: !cadastro }" class="botaoCadastro">Cadastro</ButtonChild>
        <ButtonChild @clique="selecionarLogin()" :class="{ inativo: !login }" class="botaoLogin">Login</ButtonChild>
      </div>

      <!-- Formulário de Cadastro -->
      <form class="cadastro" v-show="cadastro" @submit.prevent="cadastrarUsuario">
        <input type="email" placeholder="E-mail" required v-model="formulario.email" />
        <input type="text" placeholder="Nome de usuário" required v-model="formulario.nome" />
        <input type="password" placeholder="Senha" required v-model="formulario.senha" />
        <button type="submit" class="botaoLog">Cadastrar</button>
      </form>

      <!-- Formulário de Login -->
      <form class="login" v-show="login" @submit.prevent="logarUsuario">
        <input type="email" placeholder="E-mail" required v-model="formulario.email" />
        <input type="password" placeholder="Senha" required v-model="formulario.senha" />
        <button type="submit" class="botaoLog">Logar</button>
      </form>
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
