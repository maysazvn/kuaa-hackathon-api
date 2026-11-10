<script setup>
import { computed, ref } from 'vue'
import { postagens } from '@/data/postagens'
import Postagens from '@/components/Postagens/Postagens.vue'
import { salasUsuario } from '@/data/salasUsuario'
import { loginOut } from './account/login/Loginout'
import { pegarIDUsuario } from '@/views/account/login/UserReal'
import { users } from './user/Users'
import { salas } from '@/data/salas'

const API_URL = 'http://localhost:3000'; 

function usuarioEstaNaSala(salaIdDoPost) {
  for (let sala of salasUsuario.value) {
    const idUsuario = sala.idSala || sala.id

    if (Number(idUsuario) === Number(salaIdDoPost)) {
      return true
    }
  }
  return false
}

function pegarFotoUsuario(autorID) {
  const usuarioEncontrado = users.find((usu) => {
    return Number(usu.id) === Number(autorID)
  })
  if (usuarioEncontrado && usuarioEncontrado.pfp) {
    return usuarioEncontrado.pfp
  }
  return '/pfpPlaceholder.png'
}

const postsTimeline = computed(() => {
  return postagens.value.filter((post) => {
    const inscrito = usuarioEstaNaSala(post.salaId);
    const viral = post.curtidas >= 150;
    const meuPost = post.autorID === pegarIDUsuario()

    return inscrito || viral || meuPost
  })
})

const conteudoRapido = ref('')
const salaSelecionada = ref(null)

async function criarPostRapido() {
  if (!conteudoRapido.value.trim()) {
    alert('Escreva algo antes de postar!')
  }


  let maiorId = Math.max(...postagens.value.map((item) => item.id))

  const novoPost = {
    titulo: '',
    conteudo: conteudoRapido.value,
    autorID: pegarIDUsuario(),
    data: new Date().toLocaleDateString('pt-BR'),
    id: maiorId + 1,
    salaId: Number(salaSelecionada.value),
    curtidas: 0,
    salvos: 0,
  }

 await fetch(`${API_URL}/postagens`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(novoPost),
    })

  postagens.value.unshift(novoPost)
  localStorage.setItem('postagens', JSON.stringify(postagens.value))

  conteudoRapido.value = ''
  salaSelecionada.value = null
}

const salasVisiveis = computed(() =>
  salasUsuario.value.filter(s => salas.value.some(x => x.idSala === s.idSala))
)

</script>

<template>
  <div class="container">
    <div class="postar" v-if="loginOut === 'ativo'">
      <div class="imginput">
        <img :src="pegarFotoUsuario(pegarIDUsuario())" alt="icone de perfil do usuario" />
        <input type="text" placeholder="Qual é seu tema de estudo agora?" v-model="conteudoRapido"/>
      </div>

      <div class="botao">
        <div class="botoesSala">
          <span
            v-for="sala in salasVisiveis"
            :key="sala.idSala || sala.id"
            class="salaItem"
            :class="{ ativa: salaSelecionada === (sala.idSala || sala.id) }"
            @click="salaSelecionada = sala.idSala || sala.id"
          >
            {{ sala.nome }}
          </span>
        </div>

        <div>
          <button class="botaoPostar" type="submit" @click="criarPostRapido">Postar</button>
        </div>
      </div>
    </div>

    <div class="postar" v-if="loginOut === 'inativo'">
      <div class="semPosts">
        <p>Você não tem conta para postar!</p>
      </div>
    </div>

    <div class="semPosts" v-if="postsTimeline.length === 0">
      <p>Ainda não há posts. Experimente entrar em uma sala!</p>
    </div>

    <div class="feed" v-else>
      <Postagens :posts="postsTimeline" />
    </div>
  </div>
</template>

<style scoped>
.container {
  margin: 3vw auto 0;
  padding: 0 15px;
  box-sizing: border-box;
  color: #d9d9d9;
  max-width: 1000px;
  width: 100%;
}

.postar {
  background-color: #1e1e1e;
  border: 1px solid #2d2d2d;
  border-radius: 20px;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 20px;
  box-shadow: 0 0 6px #0000002d;
  width: 100%;
  margin: 2vw auto;
}

.postar .semPosts{
  width: 100%;
  text-align: center;
  padding: 10px 0;
  box-sizing: border-box;
}

.botoeSala {
  display: flex;
  flex: 1;
  gap: 5px;
}

.imginput {
  display: flex;
  align-items: center;
  gap: 10px;
  width: 100%;
  box-sizing: border-box;
}

.imginput img {
  width: 70px;
  height: 70px;
  border-radius: 50%;
  object-fit: cover;
}

.imginput input {
  flex: 1;
  border: none;
  outline: none;
  color: #d9d9d9;
  font-size: 1rem;
  padding: 12px 0 8px 0;
}

.botao {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 10px;
  border-top: 1px solid #282828;
  padding-top: 10px;
}

.botoesSala {
  display: flex;
  gap: 8px;
  align-items: center;
  flex: 1;
  max-width: 550px;
  overflow-x: auto;
  white-space: nowrap;
  padding: 4px 0;
  scrollbar-width: thin;
  scrollbar-color: #444444 #1e1e1e;
}

.semPosts p {
  font-size: 1.5rem;
  text-align: center;
}

.salaItem {
  font-size: 0.85rem;
  font-weight: 500;
  color: #b0b0b0;
  background-color: #313131;
  padding: 5px 15px;
  border-radius: 20px;
  cursor: pointer;
  flex-shrink: 0;
}

.salaItem:hover {
  transition: 0.2s;
  opacity: 0.9;
  color: #ffffff;
}

.salaItem.ativa {
  background-color: #ffffff;
  color: #1e1e1e;
  font-weight: bold;
}

.botaoPostar {
  color: #1e1e1e;
  font-weight: bold;
  background-color: #eeeeee;
  padding: 3px 30px;
  border-radius: 15px;
}

.botaoPostar:hover {
  transition: 0.2s;
  opacity: 0.9;
  transform: scale(0.95);
}

.salas {
  background-color: #1a1a1a;
  border: 1px solid #2d2d2d;
  border-radius: 15px;
  padding: 8px;
  max-width: 300px;
  box-shadow: 0 0 15px #0101012f;
}

@media (max-width: 768px) {

    .container{
    padding: 0;
  }


  .semPosts{
    width: 200px;
  }

  .semPosts p{
    font-size: 20px;
  }

  .postar{
    width: 100%;
  }

  .btn-fixo {
    bottom: 20px !important;
    right: 16px !important;
  }

  .text-none {
    width: 50px !important;
    height: 50px !important;
  }

  .imginput img {
  width: 50px;
  height: 50px;
  border-radius: 50%;
}
}


/*
///////////////////////////////////////////// BUGS/COISAS PRA ARRUMAR /////////////////////////////////////////////
TIMELINE:
1. curtidas!!! e salvo

SALAS:
5. quando loga em uma conta e depois sai da conta, cria uma sala e sai com o nome do criador do login mesmo nao estando logado
Samuel ta resolvendo a parte de linkar os botoes de entrar na sdala

PESQUISA:
1. aparecer seu proprio usuario na pesquisa
2. Quando logado, nehuma pesquisa funciona
(Uncaught (in promise) TypeError: user.nome.toLowerCase is not a function)

USUARIOS:
1. aparecer msg de excluir usuario assim como aparece em salas

CSS:
1. responsividade

*/
</style>
