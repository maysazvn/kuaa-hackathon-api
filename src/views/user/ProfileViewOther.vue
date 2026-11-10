<script setup>
import { useRoute } from 'vue-router';
import { computed } from 'vue';
import { ref } from 'vue';
import { seguindo } from './Following';
import { loginOut } from '../account/login/Loginout';
import Postagens from '@/components/Postagens/Postagens.vue';
import { postagens } from '@/data/postagens';
import { onMounted } from 'vue';


const route = useRoute();
const API_URL = 'http://localhost:3000';

const usuario = ref(null);
const carregando = ref(true);
const estaseguindo = ref(false);

async function carregarUsuario() {
  carregando.value = true;

  try {
    const resposta = await fetch(`${API_URL}/users/${route.params.id}`);

    if (!resposta.ok) {
      throw new Error('Usuário não encontrado');
    }

    usuario.value = await resposta.json();

    const salvo = localStorage.getItem(`seguindo_${usuario.value.id}`);
    estaseguindo.value = salvo === 'true';

  } catch (error) {
    console.error('Erro ao carregar usuário:', error);
    usuario.value = null;
  } finally {
    carregando.value = false;
  }
}
onMounted(() => {
  carregarUsuario();
});

const postsUsuario = computed(() => {
  if (!usuario.value) return [];
  const listaPosts = postagens.value || postagens;
  return listaPosts.filter((post) => post.autorID === usuario.value.id);
});

const mensagemSeguir = computed(() => (estaseguindo.value ? 'Seguindo' : 'Seguir'));



async function seguir() {
  if (loginOut.value !== 'ativo') {
    alert('Faça login para seguir!');
    return;
  }

  if (!usuario.value) return;

  try {
    const novoEstado = !estaseguindo.value;

    const novosSeguidores = novoEstado
      ? usuario.value.seguidores + 1
      : usuario.value.seguidores - 1;


    const resposta = await fetch(`${API_URL}/users/${usuario.value.id}`, {
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        seguidores: novosSeguidores
      })
    });

    if (!resposta.ok) {
      throw new Error('Não foi possível atualizar os seguidores');
    }

    usuario.value = await resposta.json();

    estaseguindo.value = novoEstado;

    seguindo.value += novoEstado ? 1 : -1;

    localStorage.setItem(
      `seguindo_${usuario.value.id}`,
      String(estaseguindo.value)
    );

    localStorage.setItem('total_seguindo', seguindo.value);

  } catch (error) {
    console.error('Erro ao atualizar seguidores:', error);
    alert('Erro ao atualizar seguidores!');
  }
}

</script>

<template>

  <div class="container">
    <div v-if="usuario" class="cartaoPerfil">
      <img v-if="usuario.banner" :src="usuario.banner" class="banner" />
      <img v-if="usuario.pfp" :src="usuario.pfp" class="foto" />

   


      <div class="acoesPerfil">
        <button class="seguirUsuario" v-on:click="seguir()">{{ mensagemSeguir }}</button>
      </div>

     
     
     
      <div class="info">
        <h1>{{ usuario.nome }}</h1>
        <p>{{ usuario.desc }}</p>
      </div>

      <div>
        <ul>
          <li class="seguidores">
            <span>{{ usuario.seguidores }}</span> Seguidores
          </li>
          <li class="seguindo">
            <span>{{ usuario.seguindo }}</span> Seguindo
          </li>
        </ul>
      </div>

        <div class="salas" v-show="usuario.mostrarSala === 'sim'">
        <ul class="listaSalas">
         <li v-for="sala in salasDoUsuario" :key="sala.idSala" class="cardSala">
             <RouterLink :to="`/salas/${sala.idSala}`">
              <span class="nomesala">{{ sala.nome }}</span>
            </RouterLink>
          </li>
        </ul>
      </div>

      <div class="postagens">
        <h2>Posts</h2>
        <hr>
        <Postagens :posts="postsUsuario"></Postagens>
      </div>
    </div>


    <div v-else>
      <p>Usuário não encontrado.</p>
    </div>
  </div>


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

button.seguirUsuario {
  background-color: #f8d768;
  color: #1e1e1e;
  padding: 3px 35px;
  border-radius: 20px;
  font-weight: bold;
  cursor: pointer;
}

button.seguirUsuario:hover {
  background-color: #f8d668e7;
  transform: scale(0.97);
  transition: 0.3s;
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
.info p{
  color: #d9d9d9;
  margin: 15px 30px;
}

.info h1{
  font-weight: bold;
  font-size: 1.8rem;
}

.info p{
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
  margin: 5px;
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
  transition: .2s;
}

.nomesala {
  color: #e0e0e0;
  font-size: 0.9rem;
  font-weight: 500;
}

.postagens{
  width: 100%;
}

.postagens h2{
  font-size: 1.4rem;
  color: #d9d9d9;
  font-family: 'Prompt', sans-serif;
  font-weight: bold;
  margin: 15px;
}

hr{
  color: #444444;
  margin: 1px 1px 25px 1px;
}

@media (max-width: 768px) {

  .container{
    padding: 0 !important;
  }

  img.foto{
    width: 80px;
    height: 80px;
    top: 130px;
  }

  .editarDeletar{
    font-size: 1.2rem;
  }

  .cardSala{
    padding: 10px 15px;
  }

}
</style>
