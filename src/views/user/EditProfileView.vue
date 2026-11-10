<script setup>
import { ref, watch } from 'vue'
import { users } from '@/views/user/Users'
import { pegarIDUsuario, userReal } from '@/views/account/login/UserReal'
import ButtonChild from '@/components/ButtonChild.vue'

// import { onMounted, ref } from 'vue';
// nome usuario é o que vc quer mudar André


const meuid = pegarIDUsuario()

function sincronizarUsuarioGlobal(campo, valor) {
  const usuarioMemoria = users.find((u) => Number(u.id) === Number(meuid))
  if (usuarioMemoria) {
    usuarioMemoria[campo] = valor
  }
  const usuariosSalvos = JSON.parse(localStorage.getItem('usuarios_cadastrados') || '[]')
  const usuarioSalvo = usuariosSalvos.find((u) => Number(u.id) === Number(meuid))

  if (usuarioSalvo) {
    usuarioSalvo[campo] = valor
    localStorage.setItem('usuarios_cadastrados', JSON.stringify(usuariosSalvos))
  }
}

const nomeUsuario = ref(localStorage.getItem('user_nome') || '')
const desc = ref(localStorage.getItem('desc') || '')

watch(nomeUsuario, (novoNome) => {
  localStorage.setItem('user_nome', novoNome)
  sincronizarUsuarioGlobal('nome', novoNome)
})

watch(desc, (novaDesc) => {
  localStorage.setItem('desc', novaDesc)
  sincronizarUsuarioGlobal('desc', novaDesc)
})

const urlFoto = ref(localStorage.getItem('urlFoto') || '/pfpPlaceholder.png')
const urlBanner = ref(localStorage.getItem('urlBanner') || '/bannerPlaceholder.png')

let novaFoto = ref(null)
let novoBanner = ref(null)

function mudarFoto(event) {
  novaFoto.value = event.target.files[0]

  if (novaFoto.value) {
    const reader = new FileReader()

    reader.onload = () => {
      urlFoto.value = reader.result
      localStorage.setItem('urlFoto', reader.result)
      sincronizarUsuarioGlobal('pfp', reader.result) 
    }

    reader.readAsDataURL(novaFoto.value)
  }
}

function mudarBanner(event) {
  novoBanner.value = event.target.files[0]

  if (novoBanner.value) {
    const reader = new FileReader()

    reader.onload = () => {
      urlBanner.value = reader.result
      localStorage.setItem('urlBanner', reader.result)
      sincronizarUsuarioGlobal('banner', reader.result)
    }

    reader.readAsDataURL(novoBanner.value)
  }
}

const mostrarSala = ref(localStorage.getItem('mostrarSala?') || 'sim')

watch(mostrarSala, (novoValor) => {
  localStorage.setItem('mostrarSala?', novoValor)
  sincronizarUsuarioGlobal('mostrarSala', novoValor)
})

// APIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII


const API_URL = 'http://localhost:3000';

async function excluirUser(id) {
   await fetch(`${API_URL}/user/${id}`,{
            method: 'DELETE',
        });
}

async function pegarIDUsuario(){
  const idUser = fetch(`${API_URL}/usuario/${userNomeReal.value}`)
}

</script>

<template>
  <div class="container">
    <div class="cartaoPerfil">
      <div class="adicionarFotoBanner">
        <label for="fotoDeBanner" class="label-banner">
          <img class="previewBanner" :src="urlBanner" alt="Preview" v-if="urlBanner" />
          <span class="icone-lapis-banner"><font-awesome-icon icon="pen" /></span>
        </label>

        <input
          type="file"
          id="fotoDeBanner"
          class="fotoBanner"
          @change="mudarBanner"
          accept="image/*"
        />
      </div>

      <div class="adicionarFotoPerfil">
        <label for="fotoDePerfil" class="label-foto">
          <img class="previewFoto" :src="urlFoto" alt="Preview" v-if="urlFoto" />
          <span class="icone-lapis-foto"><font-awesome-icon icon="pen" /></span>
        </label>
        <input
          type="file"
          id="fotoDePerfil"
          class="fotoPerfil"
          @change="mudarFoto"
          accept="image/*"
        />
      </div>

      <div class="campos">
        <div class="campo-grupo">
          <label>Nome de usuário</label>
          <input
            v-model="nomeUsuario"
            class="nomeUsuario"
            placeholder="Nome de usuário"
            maxlength="25"
          />
          <span class="contador">{{ nomeUsuario ? nomeUsuario.length : 0 }}/25</span>
        </div>
        <div class="campo-grupo">
          <label> Descrição </label>
          <textarea
            v-model="desc"
            class="descricao"
            placeholder="Escreva sobre você..."
            maxlength="300"
          ></textarea>
          <span class="contador">{{ desc ? desc.length : 0 }}/300</span>
        </div>

        <div class="campo-grupo">
          <label>Mostrar Salas?</label>
          <div class="radios">
            <div class="sim">
              <input type="radio" name="mostrarSala?" value="sim" id="yes" v-model="mostrarSala" />
              <label for="yes">Sim</label>
            </div>
            <div class="nao">
              <input type="radio" name="mostrarSala?" value="nao" id="no" v-model="mostrarSala" />
              <label for="no">Não</label>
            </div>
          </div>
        </div>
      </div>

      <router-link to="/profile">
        <button>Confirmar</button>
      </router-link>
    </div>
  </div>
  <div>
  <ButtonChild @clique="excluirUser()">
    Excluir Usuário
  </ButtonChild>
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
