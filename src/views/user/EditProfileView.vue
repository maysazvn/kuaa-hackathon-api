<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import ButtonChild from '@/components/ButtonChild.vue'
import { GoToSymbol } from 'vuetify/lib/composables/goto.mjs'
import { useUser } from '../account/LoginView.vue'
const router = useRouter()
const API_URL = 'http://localhost:3000'

const usuario = ref({
  id: '',
  nome: '',
  desc: '',
  mostrarSala: 'sim',
  pfp: '/pfpPlaceholder.png',
  banner: '/bannerPlaceholder.png'
})

// Simulando a busca do usuário logado (ajuste conforme o ID salvo no login)
async function carregarPerfil() {
  try {
    const idLogado = idUser.value
    const resposta = await fetch(`${API_URL}/users/${idLogado}`)
    if (resposta.ok) {
      usuario.value = await resposta.json()
    }
  } catch (error) {
    console.error('Erro ao carregar perfil:', error)
  }
}

async function atualizarPerfil() {
  try {GoToSymbol} catch (error) {
    console.error('Erro ao atualizar:', error)
  }
}

async function excluirUser() {
  if (confirm('Tem certeza que deseja excluir sua conta?')) {
    try {
      await fetch(`${API_URL}/users/${useUser()}`, {
        method: 'DELETE',
      })
      
      localStorage.clear()
      alert('Conta excluída com sucesso.')
      router.push('/')
    } catch (error) {
      console.error('Erro ao excluir usuário:', error)
    }
  }
}

onMounted(carregarPerfil)
</script>

<template>
  <div class="container">
    <form class="cartaoPerfil" @submit.prevent="atualizarPerfil">
      <div class="campos">
        <div class="campo-grupo">
          <label>Nome de usuário</label>
          <input v-model="usuario.nome" placeholder="Nome de usuário" maxlength="25" required />
        </div>

        <div class="campo-grupo">
          <label>Descrição</label>
          <textarea v-model="usuario.desc" placeholder="Escreva sobre você..." maxlength="300"></textarea>
        </div>

        <div class="campo-grupo">
          <label>Mostrar Salas?</label>
          <div class="radios">
            <input type="radio" value="sim" id="yes" v-model="usuario.mostrarSala" />
            <label for="yes">Sim</label>
            <input type="radio" value="nao" id="no" v-model="usuario.mostrarSala" />
            <label for="no">Não</label>
          </div>
        </div>
      </div>

      <div class="col-12 d-flex gap-2">
        <button type="submit" class="btn btn-primary">Confirmar</button>
      </div>
    </form>

    <div>
      <ButtonChild @clique="$emit(excluirUser())" class="btn btn-danger">
        Excluir Usuário
      </ButtonChild>
    </div>
  </div>
</template>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.cartaoPerfil {
  background: #2b2b2b;
  border-radius: 20px;
  padding-bottom: 30px;
  position: relative;
}

.label-banner {
  display: block;
  position: relative;
  width: 100%;
  height: 180px;
  cursor: pointer;
  background-color: #2b2b2b;
  border-radius: 20px;
}

.label-banner::after{
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: #1e1e1e73;
  z-index: 1;
}

.previewBanner {
  width: 100%;
  height: 180px;
  object-fit: cover;
  border-radius: 20px;
}

.icone-lapis-banner {
  position: absolute;
  top: 50%;
  left: 50%;
  font-size: 1.8rem;
  color: #d9d9d9a5;
  padding: 10px;
  transform: translate(-50%, -50%);
  border: 3px solid #d9d9d9a5;
  border-radius: 20px;
  z-index: 10;
}

input.fotoBanner{
  display: none;
}

.adicionarFotoPerfil {
  position: relative;
  margin-top: -60px;
  margin-left: 30px;
  width: 8vw;
  height: 8vw;
  z-index: 10;
  border-radius: 50%;
}

.previewFoto {
  display: block;
  width: 8vw;
  height: 8vw;
  border-radius: 50%;
  border: 5px solid #2b2b2b;
  background-color: #2b2b2b;
  cursor: pointer;
  position: relative;
}

.adicionarFotoPerfil::after{
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: #1e1e1e73;
  z-index: 1;
  border-radius: 50%;

}

.icone-lapis-foto {
  position: absolute;
  top: 50%;
  left: 50%;
  font-size: 0.9rem;
  color: #d9d9d9a5;
  padding: 5px;
  transform: translate(-50%, -50%);
  border: 3px solid #d9d9d9a5;
  border-radius: 10px;
  z-index: 10;
  cursor: pointer;
}

input.fotoPerfil{
  display: none;
}

.campos {
  padding: 0 30px;
  margin-top: 20px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.campo-grupo {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.campo-grupo label {
  font-size: 1.1rem;
  font-weight: bold;
  color: #d9d9d9;
}

.nomeUsuario {
  border: 2px solid #3c3c3c;
  border-radius: 10px;
  padding: 6px 10px;
  color: #8f8f8f;
  width: 250px;
  outline: none;
}

.descricao {
  border: 2px solid #3c3c3c;
  border-radius: 10px;
  padding: 6px 10px;
  color: #8f8f8f;
  width: 100%;
  max-width: 500px;
  height: 100px;
  resize: none;
  outline: none;
}

.nomeUsuario:focus,
.descricao:focus {
  border: 2px solid #f8d66888;
}

.contador {
  color: #8f8f8f;
  font-size: 0.8rem;
}

.radios {
  display: flex;
}

.sim,
.nao {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-right: 20px;
  margin-top: 5px;
  color: #d9d9d9;
  font-weight: bold;
}

.sim input[type='radio'],
.nao input[type='radio'] {
  accent-color: #f8d668;
  cursor: pointer;
}

.sim label,
.nao label {
  cursor: pointer;
  font-size: 0.95rem;
  color: #d9d9d9;
}

button {
  background: #f8d668;
  color: #1e1e1e;
  border: none;
  padding: 5px 15px;
  border-radius: 20px;
  font-weight: bold;
  cursor: pointer;
  margin: 20px;
}

button:hover {
  background-color: #f8d668e7;
  transform: scale(0.97);
  transition: 0.3s;
}

@media (max-width: 768px) {

  .container {
    padding: 0 !important;
  }

  .label-banner {
    height: 140px;
  }

  .previewBanner {
    height: 140px;
  }

  .adicionarFotoPerfil {
    width: 90px;
    height: 90px;
    margin-left: 20px;
    margin-top: -45px;
  }

  .previewFoto {
    width: 90px;
    height: 90px;
  }

  .campos {
    padding: 0 20px;
  }

  .nomeUsuario {
    width: 100%;
  }

  .descricao {
    max-width: 100%;
  }

  button {
    width: calc(100% - 40px);
    margin: 20px;
  }

}

@media (min-width: 769px) and (max-width: 1024px) {
  .adicionarFotoPerfil,
  .previewFoto {
    width: 100px;
    height: 100px;
  }
}
</style>
