<script setup>
import { useRoute, useRouter } from 'vue-router'
import { computed, ref, onMounted } from 'vue'
import { loginOut } from '../account/login/Loginout'
import { userReal } from '../account/login/UserReal'
import ButtonEnter from '@/components/ButtonEnter.vue'
import Postagens from '@/components/Postagens/Postagens.vue'
import { postagens } from '@/data/postagens'
import { users } from '../user/Users'

const API_URL = 'http://localhost:3000'

const route = useRoute()
const router = useRouter()

const sala = ref(null)
const popupExcluir = ref(false)
const menuAberto = ref(false)

async function buscarSala() {
  try {
    const resposta = await fetch(`${API_URL}/salas/${route.params.id}`)

    if (!resposta.ok) {
      sala.value = null
      return
    }

    sala.value = await resposta.json()
  } catch (erro) {
    console.error('Erro ao buscar sala:', erro)
  }
}

onMounted(buscarSala)

const postsSala = computed(() => {
  if (!sala.value) return []

  return postagens.value.filter(
    post => post.salaId == sala.value.id
  )
})

const criador = computed(() => {
  return users.find(
    usuario => usuario.nome === sala.value?.usuarioCriador
  )
})

const statusTexto = computed(() => {
  return sala.value?.status == 1 ? 'Ativo' : 'Inativo'
})

function excluirSala() {
  popupExcluir.value = true
}

async function confirmarEx() {
  try {
    const resposta = await fetch(`${API_URL}/salas/${sala.value.id}`, {
      method: 'DELETE'
    })

    if (!resposta.ok) {
      throw new Error('Erro ao excluir sala')
    }

    popupExcluir.value = false
    router.push('/explore')
  } catch (erro) {
    console.error(erro)
    alert('Não foi possível excluir a sala.')
  }
}

function editarSala() {
  router.push(`/salas/${route.params.id}/editar`)
}
</script>
<template>
  <div class="container-sala" v-if="sala">
    <div class="header-pagina">
      <div class="titulo">
        <RouterLink to="/explore" class="voltar">
          <font-awesome-icon icon="chevron-left"></font-awesome-icon> Perfil de sala</RouterLink
        >
      </div>
    </div>

    <div class="card-sala">
      <div class="topo-sala">
        <div class="fotoinfo">
        <img :src="sala.banner" class="foto-sala" />

        <div class="info-sala">
          <h2 class="nome-sala">{{ sala.nome }}</h2>
          <div class="metadados">
            <div>
              <RouterLink v-if="sala.usuarioCriador === userReal" :to="`/profile`">
                <span>CRIADOR</span> {{ sala.usuarioCriador }}
              </RouterLink>
              <RouterLink v-else :to="`/otherProfile/${criador.id}`">
                <span>CRIADOR</span> {{ sala.usuarioCriador }}
              </RouterLink>
            </div>

            <p><span>STATUS</span> {{ statusTexto }}</p>
            </div>
          </div>
        </div>

        <div class="acoes-sala">
          <ButtonEnter :sala="sala" v-if="sala.usuarioCriador != userReal" />
          <div class="menu">
          
          

            <div class="menu" v-if="sala.usuarioCriador === userReal">
              <button class="menubotao" @click="menuAberto = !menuAberto">...</button>
              <div class="menuaberto" v-if="menuAberto">
                <button @click="editarSala">Editar sala</button>
                <button @click="excluirSala">Apagar sala</button>
              </div>
            </div>
          </div>
        </div>
      </div>  

        <p class="descricao">{{ sala.desc }}</p>

        <div class="membros">
          <span>{{ sala.participantes }} membros</span>
        </div>

        <div class="secao-posts">
          <h2>Posts</h2>
          <hr />
          <Postagens :posts="postsSala"></Postagens>
        </div>
      

      <div v-if="popupExcluir" class="telapopup">
        <div class="popup">
          <h2>Tem certeza que deseja excluir sua sala?</h2>
          <p>Esta ação é permanente e todos os seus dados e posts serão perdidos para sempre.</p>
          <div class="botoes">
            <button @click="confirmarEx" class="btn-confirmar">Apagar</button>
            <button @click="popupExcluir = false" class="btn-cancelar">Cancelar</button>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div class="container-sala" v-else>
    <h1>Sala não encontrada</h1>
  </div>
</template>
<style scoped>
.container-sala {
  max-width: 850px;
  margin: 0 auto;
  padding: 20px;
  color: #d9d9d9;
}

.header-pagina {
  margin-bottom: 25px;
}

a.voltar {
  font-size: 2rem;
  font-family: 'Prompt', sans-serif;
  font-weight: bold;
  color: #d9d9d9;
}

.titulo {
  display: flex;
  align-items: center;
}

.topo-sala {
  display: flex;
  gap: 20px;
  position: relative;
  align-items: center;
  justify-content: space-between;
}

.foto-sala {
  width: 90px;
  height: 90px;
  border-radius: 15px;
  object-fit: cover;
}

.info-sala {
  flex: 1;
}

.nome-sala {
  font-size: 1.8rem;
  font-weight: 800;
  color: #d9d9d9;
  margin: 0 0 10px 0;
}

.metadados {
  display: flex;
  flex-direction: column;
  gap: 5px;
  font-size: 1rem;
  font-family: 'Prompt', sans-serif;
}

.metadados span {
  color: #888888;
  font-weight: bold;
  margin-right: 8px;
}

.acoes-sala {
  display: flex;
  align-items: center;
  gap: 10px;
}

.btn-entrar {
  background-color: #f8d668;
  color: #1e1e1e;
  border: none;
  border-radius: 20px;
  padding: 5px 25px;
  font-weight: bold;
  cursor: pointer;
}

.btn-entrar span{
  font-weight: bold;
}

.btn-entrar:hover {
  opacity: 0.9;
  transform: scale(0.95);
  transition: 0.2s;
}

.btn-entrar.btn-sair {
  background-color: #3c3c3c;
  color: #d9d9d9;
}

.btn-entrar.btn-sair:hover {
  background-color: #e53935;
  color: #ffffff;
  border-color: #e53935;
}

.menu {
  position: relative;
}

.menubotao {
  color: #d9d9d9;
  font-size: 1.5rem;
  cursor: pointer;
  padding: 0 5px;
}

.menuaberto {
  position: absolute;
  top: 45x;
  background-color: #313131;
  color: #d9d9d9;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.179);
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  padding: 5px;
  z-index: 10;
  width: 110px;
}

.menuaberto button:hover {
  color: #f8d668;
  transform: scale(0.97);
  transition: 0.3s;
}

.descricao {
  margin-top: 20px;
  line-height: 1.2;
  max-width: 600px;
  word-break: break-word;
  font-size: 1.1rem;
}

.membros {
  display: flex;
  align-items: center;
  gap: 5px;
  margin-top: 15px;
  color: #aaaaaa;
  font-size: 1rem;
}

.secao-posts{
  padding: 0;
}

.secao-posts h2 {
  font-size: 1.4rem;
  color: #d9d9d9;
  font-family: 'Prompt', sans-serif;
  font-weight: bold;
  margin: 25px 0 5px 0;
}

.botoes {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 15px;
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

.botoes {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 15px;
  width: 100%;
}

hr{
  color: #333333;
  margin: 1px 1px 25px 1px;
}

.fotoinfo{
  display: flex;
    gap: 15px;
    align-items: center;
  }

@media (max-width: 768px) {

  .topo-sala{
    display: flex;
    flex-direction: column;
  }

  .fotoinfo{
    display: flex;
    align-items: center;
    gap: 15px;
  }

}
</style>
