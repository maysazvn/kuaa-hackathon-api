<script setup>
// imports //////////////////

import { loginOut } from '@/views/account/login/Loginout'
import { ref } from 'vue'
import { RouterLink } from 'vue-router'
import { salasUsuario } from '@/data/salasUsuario'
import { onMounted } from 'vue'
import { salas } from '@/data/salas'
import { userReal } from '@/views/account/login/UserReal'
import { emailReal } from '@/views/account/login/EmailReal'
import { urlFoto } from '@/views/user/urlFoto'
const menuSalasAberto = ref(false);
const menuMaisAberto = ref(false)

import { computed } from 'vue'

 const salasVisiveis = computed(() =>
  salasUsuario.value.filter(s => salas.value.some(x => x.idSala === s.idSala))
)

// functions /////////////////
function validarLoginout() {
  if (loginOut.value == 'ativo') {
    loginOut.value = 'processo'
    localStorage.setItem('login_status', 'inativo')
  } else {
    alert('Faça login primeiro!')
  }
}

onMounted(() => {
   const statusLogin = localStorage.getItem('login_status')
  if (statusLogin === 'ativo') {
    loginOut.value = 'ativo'
    userReal.value = localStorage.getItem('user_nome')
    emailReal.value = localStorage.getItem('user_email')
    
    
  }
})
</script>

<template>
  <aside class="sidebar">
    <nav class="geral">
      <ul>
        <li>
          <router-link to="/"> <font-awesome-icon icon="house" /> Página Inicial </router-link>
        </li>
        <li>
          <router-link to="/explore"> <font-awesome-icon icon="compass" /> Explorar </router-link>
        </li>
        <li>
          <router-link to="/profile"> <font-awesome-icon icon="user" /> Perfil </router-link>
        </li>
        <router-link to="/login"></router-link>
      </ul>
    </nav>

    <hr />

    <nav class="salas">
      <div class="criarSala">
        <div class="criar">
          <h1>SALAS</h1>
          <router-link to="/room"> <font-awesome-icon icon="plus" class="mais" /> </router-link>
          <!-- esse font-awesome-icon é o botão de mais ao lado do título salas que vai abrir o adicionar sala -->
        </div>

        <div class="salas">
          <div v-for="sala in salasVisiveis" :key="sala.nome" class="salas-link">
            <RouterLink :to="`/salas/${sala.idSala}`">
              <h2 class="sala-link">{{ sala.nome }}</h2>
            </RouterLink>
          </div>
          <!-- quem for criar as salas, já cria lá na pasta data um dataset de salas padrão pfv, pra deixar um v-for aqui. -->
        </div>
      </div>
    </nav>

    <hr />

    <nav class="suporte">
      <h1>SUPORTE</h1>
      <ul>
        <li>
          <router-link to="/help"> <font-awesome-icon icon="circle-question" /> Ajuda </router-link>
        </li>
        <li>
          <router-link to="/about"> <font-awesome-icon icon="circle-info" /> Sobre </router-link>
        </li>
      </ul>
    </nav>

    <div class="dadosUser" v-if="loginOut == 'ativo'">
      <div>
        <img :src="urlFoto" alt="Foto de perfil do usuário" class="imgSidebar" />
      </div>

      <div>
        <p class="usuarioSidebar">{{ userReal }}</p>
        <p class="emailSidebar">{{ emailReal }}</p>
      </div>
    </div>

    <div class="sairUsuario" v-if="loginOut == 'ativo'">
      <span @click="validarLoginout()" class="sair"
        ><font-awesome-icon icon="arrow-right-from-bracket" class="sair" /> Sair</span
      >
    </div>

    <div class="entrarUsuario" v-else>
      <span>
        <router-link to="/login" class="entrar"
          ><font-awesome-icon icon="arrow-right-from-bracket" class="sair" />Entrar</router-link
        >
      </span>
    </div>
  </aside>

  <nav class="bottom">
    <router-link to="/" class="item">
      <font-awesome-icon icon="house" />
      <span>Início</span>
    </router-link>
    <router-link to="/explore" class="item">
      <font-awesome-icon icon="compass" />
      <span>Explorar</span>
    </router-link>
    <router-link to="/profile" class="item">
      <font-awesome-icon icon="user" />
      <span>Perfil</span>
    </router-link>
    <div class="item item-salas" @click="menuSalasAberto = !menuSalasAberto; menuMaisAberto = false">
      <font-awesome-icon icon="comments" />
      <span>Salas</span>

      <div class="salasAberto" v-if="menuSalasAberto" @click.stop>
        <RouterLink to="/room" class="sala-link criar-sala-link">
          <font-awesome-icon icon="plus" />
          <span>Criar sala</span>
        </RouterLink>

        <div v-for="sala in salasUsuario" :key="sala.idSala || sala.nome">
          <RouterLink :to="`/salas/${sala.idSala}`" class="sala-link">
            <span>{{ sala.nome }}</span>
          </RouterLink>
        </div>
      </div>
    </div>

   <div class="item item-mais" @click="menuMaisAberto = !menuMaisAberto; menuSalasAberto = false">
  <font-awesome-icon icon="ellipsis" />
  <span>Mais</span>

  <div class="maisAberto" v-if="menuMaisAberto" @click.stop>
    <RouterLink to="/help" class="sala-link">
      <font-awesome-icon icon="circle-question" />
      <span>Ajuda</span>
    </RouterLink>
    <RouterLink to="/about" class="sala-link">
      <font-awesome-icon icon="circle-info" />
      <span>Sobre</span>
    </RouterLink>
  </div>
</div>
  </nav>
</template>

<style scoped>
aside.sidebar {
  position: fixed;
  top: 60px;
  left: 0;
  bottom: 0;
  z-index: 999;
  background: #313131;
  display: flex;
  flex-direction: column;
  width: 250px;
  padding: 10px 5px;
  box-sizing: border-box;
}

.bottom {
  display: none;
}

h1 {
  color: #d9d9d9;
  font-weight: bold;
  margin: 5px 0;
  font-size: 1.5rem;
}

nav.salas {
  display: flex;
  flex-direction: column;
}

.criar {
  color: #d9d9d9;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 5px;
}

div.salas {
  color: #d9d9d9;
}

ul li a {
  color: #d9d9d9;
  font-weight: 600;
  font-size: 1.3rem;
  transition: 0.25s ease;
}

ul li:hover {
  transform: scale(0.95);
  transition: 0.2s;
}

ul li a.router-link-active {
  border-left: 3px solid #f8d76b;
  padding: 3px 15px 3px 6px;
  border-radius: 5px;
  box-shadow: inset 4px 0 8px -2px #a98e3641;
}

.mais {
  margin: 8px;
}

.router-link-active .mais {
  transform: scale(1.4);
  transition: 0.2s;
}

hr {
  color: #d9d9d9;
  margin: 2px;
}

.icone-grupo {
  right: 0% !important;
}

.dadosUser {
  display: flex;
  color: #d9d9d9;
  justify-content: baseline;
  align-items: center;
}

.usuarioSidebar,
.emailSidebar {
  margin: 1px 10px;
  white-space: nowrap;
overflow: hidden;
text-overflow: ellipsis;
max-width: 140px;
}

.usuarioSidebar {
  font-weight: bold;
}

.imgSidebar {
  width: 60px;
  height: 60px;
  border-radius: 100px;
}

.sair,
.entrar {
  color: #d9d9d9;
  font-size: 1.2rem;
  font-weight: 500;
  gap: 5px;
  display: inline-flex;
  align-items: center;
  gap: 10px;
}

.sair:hover,
.entrar:hover {
  opacity: 0.8;
  transition: 0.2s;
}

.entrarUsuario,
.dadosUser {
  margin-top: auto;
}

.entrarUsuario,
.sairUsuario,
.dadosUser {
  padding: 2px 10px;
}

.sala-link{
  font-size: 1.2rem;
}

.sala-link:hover{
color: #e1bc42;
transition: .2s;
transform: scale(0.95);
}


@media (max-width: 768px) {
  aside.sidebar {
    display: none;
  }

  .bottom {
    display: flex;
    justify-content: space-around;
    align-items: center;
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    background: #313131;
    padding: 10px 0;
    z-index: 999;
    color: #d9d9d9;
  }

  .bottom .item {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 4px;
    color: #d9d9d9;
    text-decoration: none;
    font-size: 0.75rem;
    cursor: pointer;
  }

  .item-salas, .item-mais {
    position: relative;
  }

  .salasAberto, .maisAberto {
    position: absolute;
    bottom: 45px;
    right: 0;
    background-color: #1a1a1a;
    border: 1px solid #333333;
    border-radius: 10px;
    padding: 5px;
    min-width: 120px;
    max-height: 200px;
    overflow-y: auto;
    box-shadow: 0 0 15px rgba(0, 0, 0, 0.308);
    display: flex;
    flex-direction: column;
    gap: 4px;
    z-index: 1000;
  }

  .sala-link {
    display: flex;
    align-items: center;
    gap: 5px;
    padding: 8px 10px;
    color: #ffffff;
    text-decoration: none;
    font-weight: 600;
    font-size: 0.85rem;
    border-bottom: 1px solid #2a2a2a;
    white-space: nowrap;
  }

  .salasAberto div:last-child .sala-link,
  .maisAberto div:last-child .sala-link {
    border-bottom: none;
  }

  .criar-sala-link {
    color: #f8d76b;
  }
}
</style>
