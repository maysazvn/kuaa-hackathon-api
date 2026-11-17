<!-- read usuario -->
<script setup>
import { ref, computed } from 'vue'
/* import { novaFoto } from '../user/EditProfileView.vue'; */
import { seguidores } from './Followers'
import { seguindo } from './Following'
import { userReal } from '../account/login/UserReal'
import { urlFoto } from './urlFoto'
import { salas } from '@/data/salas'
import { loginOut } from '../account/login/Loginout'
import { pegarIDUsuario } from '../account/login/UserReal'
import { postagens } from '@/data/postagens'
import Postagens from '@/components/Postagens/Postagens.vue'

const suarios = JSON.parse(localStorage.getItem('salasEntradas')) || []
const nomeUsuario = ref(localStorage.getItem('nomeUsuario') || userReal)
const desc = ref(localStorage.getItem('desc') || '')
const urlBanner = ref(localStorage.getItem('urlBanner') || '/bannerPlaceholder.png')
const mostrarSala = ref(localStorage.getItem('mostrarSala?') || 'sim')
const abaAtiva = ref('posts')

const postsSalvos = computed(() => {
  return postagens.value.filter((post) => post.salvou === true)
})

const postsExibidos = computed(() => {
  if (abaAtiva.value === 'posts') {
    return postsUsuario.value
  } else {
    return postsSalvos.value
  }
})
const popupExcluir = ref(false)

const postsUsuario = computed(() => {
  return postagens.value.filter((post) => post.autorID === pegarIDUsuario())
})

function buscarSalas() {
  return salas.value.filter((sala) => {
    for (const item of suarios) {
      if (item.idSala === sala.idSala) {
        return true
      }
    }
    return false
  })
}

let existe = ref(true)
// const urlFoto = ref(localStorage.getItem('urlFoto') || '');

let mostrar = ref(false)

function mostrarItens() {
  mostrar.value = !mostrar.value
  console.log(mostrar)
}

function editar() {
  console.log('editando...')
}

function excluir() {
  localStorage.clear()
  existe.value = false
  popupExcluir.value = true
}

function excluirUser() {
  popupExcluir.value = true
}
</script>

<template>
  <span v-if="loginOut === 'ativo'">
    <div class="container" v-show="existe == true">
      <div class="cartaoPerfil">
        <img v-if="urlBanner" :src="urlBanner" class="banner" />
        <img v-if="urlFoto" :src="urlFoto" class="foto" />

        <div class="acoesPerfil">
          <div class="menu">
            <button class="editarDeletar" v-on:click="mostrarItens()">•••</button>

            <div class="editEdelete" v-show="mostrar == true" v-on:click="mostrarItens()">
              <router-link to="/edit">
                <button v-on:click="editar" class="btn-menu">Editar</button>
              </router-link>
              <!-- <button v-on:click="excluirUser" class="btn-menu">Deletar</button> -->
            </div>

            <!-- <div v-if="popupExcluir" class="telapopup">
              <div class="popup">
                <h2>Tem certeza que deseja excluir seu usuário?</h2>
                <p>
                  Esta ação é permanente e todos os seus dados, salas e posts serão perdidos para
                  sempre.
                </p>
                <div class="botoes">
                  <button @click="excluir" class="btn-confirmar">Apagar</button>
                  <button @click="popupExcluir = false" class="btn-cancelar">Cancelar</button>
                </div>
              </div>
            </div> -->
          </div>
        </div>

        <div class="info">
          <h1>{{ nomeUsuario }}</h1>
          <p>{{ desc }}</p>
        </div>

        <div>
          <ul>
            <li class="seguidores">
              <span>{{ seguidores }}</span> Seguidores
            </li>
            <li class="seguindo">
              <span>{{ seguindo }}</span> Seguindo
            </li>
          </ul>
        </div>
      </div>

      <div class="salas" v-show="mostrarSala === 'sim'">
        <ul class="listaSalas">
          <li v-for="sala in buscarSalas()" :key="sala.idSala" :nome="sala.nome" class="cardSala">
            <RouterLink :to="`/salas/${sala.idSala}`">
              <span class="nomesala">{{ sala.nome }}</span>
            </RouterLink>
          </li>
        </ul>
      </div>

      <div class="tabs">
        <button :class="{ ativo: abaAtiva === 'posts' }" @click="abaAtiva = 'posts'">Posts</button>
        <button :class="{ ativo: abaAtiva === 'salvos' }" @click="abaAtiva = 'salvos'">
          Salvos
        </button>
      </div>

      <div class="postagens">
        <h2>Posts</h2>
        <hr />
        <Postagens :posts="postsExibidos"></Postagens>
      </div>
    </div>
  </span>
  <span v-else>
    <p class="login">Faça login para editar seu perfil!</p>
  </span>
</template>

<style scoped>
.container {
  max-width: 900px;
  margin: 0 auto;
  padding: 20px;
}

.cartaoPerfil {
  position: relative;
  width: 100%;
  margin-bottom: 60px;
}

.acoesPerfil {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  gap: 12px;
  margin-top: 15px;
}

.menu {
  position: relative;
}

.banner {
  width: 100%;
  height: 180px;
  object-fit: cover;
  border-radius: 16px;
}

.foto {
  width: 110px;
  height: 110px;
  object-fit: cover;
  border-radius: 50%;
  position: absolute;
  z-index: 10;
  top: 130px;
  left: 25px;
  border: 5px solid #1e1e1e;
  background-color: #1e1e1e;
}

.editarDeletar {
  color: #d9d9d9;
}

div.editEdelete {
  position: absolute;
  right: 0;
  top: 35px;
  background-color: #313131;
  color: #d9d9d9;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.179);
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  padding: 5px;
  z-index: 10;
}

.editEDelete a button {
  padding: 5px 10px;
  text-align: left;
  cursor: pointer;
}

:deep(.btn-menu:hover) {
  color: #f8d668;
  transform: scale(0.97);
  transition: 0.3s;
}

ul {
  list-style: none;
  display: flex;
  gap: 20px;
  margin: 15px 0;
}

.info {
  padding-top: 15px;
}

.info h1,
.info p {
  color: #d9d9d9;
  margin: 15px 30px;
}

.info h1 {
  font-weight: bold;
  font-size: 1.8rem;
}

.info p {
  max-width: 500px;
  word-break: break-word;
}

.seguidores,
.seguindo {
  font-size: 0.95rem;
  color: #d9d9d9;
  margin: 15px 30px;
}

.seguidores span,
.seguindo span {
  font-weight: bold;
}

.listaSalas {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  gap: 10px;
  overflow-x: auto;
  white-space: nowrap;
  padding-bottom: 6px;
  scrollbar-width: thin;
  scrollbar-color: #444444 #1e1e1e;
}

.cardSala {
  display: flex;
  align-items: center;
  gap: 5px;
  background-color: #313131;
  padding: 1px 10px;
  border-radius: 15px;
  cursor: pointer;
  color: #d9d9d9;
}

.cardSala:hover {
  opacity: 0.9;
  transform: scale(0.95);
  transition: 0.2s;
}

.nomesala {
  color: #e0e0e0;
  font-size: 0.9rem;
  font-weight: 500;
}

.postagens {
  width: 100%;
}

.postagens h2 {
  font-size: 1.4rem;
  color: #d9d9d9;
  font-family: 'Prompt', sans-serif;
  font-weight: bold;
  margin: 15px;
}

hr {
  color: #444444;
  margin: 1px 1px 25px 1px;
}

.tabs {
  display: flex;
  gap: 10px;
  margin: 20px 0;
}

.tabs button {
  background: transparent;
  color: #8f8f8f;
  border: none;
  padding: 8px 16px;
  font-weight: bold;
  cursor: pointer;
  border-bottom: 2px solid transparent;
}

.tabs button.ativo {
  color: #f8d76b;
  border-bottom: 2px solid #f8d76b;
}

.login {
  text-align: center;
  color: #d9d9d9;
  font-size: 2rem;
  font-weight: bold;
}

.btn-confirmar,
.btn-cancelar {
  padding: 5px 32px;
  border-radius: 20px;
  border: none;
  font-weight: bold;
  font-size: 1rem;
  cursor: pointer;
}

.btn-confirmar:hover,
.btn-cancelar:hover {
  opacity: 0.9;
  transform: scale(0.97);
  transition: 0.3s;
}

.btn-confirmar {
  background-color: #7e7e7e;
  color: #1e1e1e;
}

.btn-cancelar {
  background-color: #f8d668;
  color: #000000;
}

.telapopup {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: #00000065;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.popup {
  background-color: #242424;
  border: 1px solid #333333;
  padding: 30px;
  border-radius: 15px;
  max-width: 420px;
  width: 90%;
  text-align: center;
  box-shadow: 0px 10px 30px #00000037;
}

.popup h2 {
  font-size: 1.1rem;
  margin-bottom: 10px;
  font-weight: bold;
}

.popup p {
  font-size: 1rem;
  color: #a5a5a5;
  margin-bottom: 24px;
}

.botoes{
  display: flex;
  gap: 15px;
  justify-content: center;
}
@media (max-width: 768px) {
  .container {
    padding: 0 !important;
  }

  img.foto {
    width: 80px;
    height: 80px;
    top: 130px;
  }

  .editarDeletar {
    font-size: 1.2rem;
  }

  .cardSala {
    padding: 10px 15px;
  }
}
</style>
