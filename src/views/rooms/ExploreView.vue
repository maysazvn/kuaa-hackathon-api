<script setup>
import { ref, onMounted } from 'vue'
import CartSala from '@/components/cart/CartSala.vue'
import ButtonEnter from '@/components/ButtonEnter.vue';
import { loginOut } from '../account/login/Loginout';
import { userReal } from '../account/login/UserReal';

// Lista reativa que vai armazenar as salas vindas da API
const salas = ref([])

// Buscar todas as salas com GET quando o componente carregar
onMounted(async () => {
  try {
    const resposta = await fetch('http://localhost:3000/salas')
    if (resposta.ok) {
      salas.value = await resposta.json()
    } else {
      console.error('Erro ao carregar as salas')
    }
  } catch (error) {
    console.error('Erro de conexão com o servidor:', error)
  }
})
</script>

<template>
  <div class="container">
    <div class="titulo">
      <RouterLink to="/" class="voltar"> <font-awesome-icon icon="chevron-left"></font-awesome-icon> Explorar salas</RouterLink>
    </div>
    <section class="salas">
      <CartSala
        v-for="sala in salas"
        :key="sala.id"
        :idSala="sala.id"
        :nome="sala.nome"
        :participantes="sala.participantes"
        :desc="sala.desc"
        :usuarioCriador="sala.usuarioCriador"
        :banner="sala.banner"
        :status="sala.status"
      >
        <div class="nav">
          <RouterLink :to="`/salas/${sala.id}`" class="visualizar"> Visualizar </RouterLink>
          <ButtonEnter :sala="sala" v-if="sala.usuarioCriador != userReal" />
        </div>
      </CartSala>
    </section>
  </div>
</template>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
}

a.voltar{
  font-size: 2rem;
  font-family: 'Prompt', sans-serif;
  font-weight: bold;
  color: #E0D8C3;
}

.titulo {
  display: flex;
  align-items: center;
}

.salas {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 2vw;
  margin: 0px auto;
  padding: 40px;
  font-size: 1.2rem;
}

.nav{
  display: flex;
  justify-content: center;
  gap: 5px;
  padding: 5px;
}

a.visualizar{
  background: #F8D76B;
  font-weight: bold;
  color: #1e1e1e;
}

.btn-entrar {
  background: #F8D76B;
  color: #1e1e1e;
}

.btn-sair {
  background-color: #3c3c3c;
  color: #d9d9d9;
}

.btn-sair:hover {
  background-color: #e53935;
  color: #ffffff;
  border-color: #e53935;
}

a.visualizar,
.btn-entrar,
.btn-sair {
  font-weight: bold;
  font-size: 0.95rem;
  padding: 5px 15px;
  border-radius: 25px;
  border: none;
  cursor: pointer;
  text-decoration: none;
  transition: all 0.2s ease;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

button{
  background: #F8D76B;
  font-weight: bold;
  border-radius: 10px;
  padding: 1px 10px;
}

a.visualizar:hover{
  opacity: 0.9;
  transform: scale(0.95);
  transition: .2s;
}

button:hover{
  opacity: 0.9;
  transform: scale(0.95);
  transition: .2s;
}

.container {
  text-align: center;
}

@media (max-width: 768px) {

  .salas{
    padding: 20px 15px;
  }

  .nav{
    display: flex;
    flex-direction: column;
    gap: 10px;
    justify-content: center;
    margin: 0 auto;
    align-items: center;
  }

  a.visualizar, .btn-entrar{
    width: 200px;
    padding: 10px 12px;
  }

}

</style>
