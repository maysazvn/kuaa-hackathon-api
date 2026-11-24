<script setup>
import { ref, onMounted, watch } from 'vue'
import { loginOut } from '@/views/account/login/Loginout'
import { userReal } from '@/views/account/login/UserReal'

const API_URL = 'http://localhost:3000'

const props = defineProps({
  sala: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['atualizar-participantes'])

const participando = ref(false)
const carregando = ref(false)


function obterSalaId() {
  const id = props.sala?.id ?? props.sala?.idSala
  if (!id || String(id) === 'undefined') return null
  return String(id)
}


function obterUsuario() {
  const u = typeof userReal === 'object' && userReal !== null ? userReal.value : userReal
  return u ? String(u).trim() : ''
}


async function verificarParticipacao() {
  const salaIdReal = obterSalaId()
  const usuarioAtual = obterUsuario()

  if (!salaIdReal || !usuarioAtual) {
    participando.value = false
    return
  }

  try {
    const resposta = await fetch(`${API_URL}/salasUsuario`)
    if (!resposta.ok) return

    const todosVinculos = await resposta.json()
    const jaEsta = todosVinculos.some(
      (item) => String(item.salaId) === salaIdReal && String(item.usuario) === usuarioAtual
    )

    participando.value = jaEsta
  } catch (erro) {
    console.error('Erro ao verificar participação:', erro)
  }
}

async function entrar() {
  const statusLogin = typeof loginOut === 'object' && loginOut !== null ? loginOut.value : loginOut
  if (statusLogin !== 'ativo') {
    alert('Faça login para entrar!')
    return
  }

  const salaIdReal = obterSalaId()
  const usuarioAtual = obterUsuario()

  if (!salaIdReal || !usuarioAtual) {
    alert('Erro: ID da sala ou usuário inválido.')
    return
  }

  if (carregando.value) return
  carregando.value = true

  try {

    const resCheck = await fetch(`${API_URL}/salasUsuario`)
    const todosVinculos = await resCheck.json()
    const jaExiste = todosVinculos.some(
      (item) => String(item.salaId) === salaIdReal && String(item.usuario) === usuarioAtual
    )

    if (jaExiste) {
      participando.value = true
      return
    }

   
    const resposta = await fetch(`${API_URL}/salasUsuario`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        salaId: salaIdReal,
        usuario: usuarioAtual
      })
    })

    if (!resposta.ok) throw new Error('Erro ao registrar entrada')


    const resSala = await fetch(`${API_URL}/salas/${salaIdReal}`)
    const salaAtual = await resSala.json()
    const novaQuantidade = (Number(salaAtual.participantes) || 0) + 1


    const resPatch = await fetch(`${API_URL}/salas/${salaIdReal}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        participantes: novaQuantidade
      })
    })

    if (!resPatch.ok) throw new Error('Erro ao atualizar participantes')

    props.sala.participantes = novaQuantidade
    emit('atualizar-participantes', novaQuantidade)
    participando.value = true

  } catch (erro) {
    console.error(erro)
    alert('Não foi possível entrar na sala.')
  } finally {
    carregando.value = false
  }
}

async function sair() {
  const salaIdReal = obterSalaId()
  const usuarioAtual = obterUsuario()

  if (!salaIdReal || !usuarioAtual || carregando.value) return
  carregando.value = true

  try {

    const resCheck = await fetch(`${API_URL}/salasUsuario`)
    const todosVinculos = await resCheck.json()
    const vinculosParaApagar = todosVinculos.filter(
      (item) => String(item.salaId) === salaIdReal && String(item.usuario) === usuarioAtual
    )

    for (const vinculo of vinculosParaApagar) {
      const delRes = await fetch(`${API_URL}/salasUsuario/${vinculo.id}`, {
        method: 'DELETE'
      })
      if (!delRes.ok) throw new Error(`Erro ao apagar vínculo ${vinculo.id}`)
    }

    if (vinculosParaApagar.length > 0) {
      const resSala = await fetch(`${API_URL}/salas/${salaIdReal}`)
      const salaAtual = await resSala.json()
      const novaQuantidade = Math.max(0, (Number(salaAtual.participantes) || 0) - 1)

      const resPatch = await fetch(`${API_URL}/salas/${salaIdReal}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          participantes: novaQuantidade
        })
      })

      if (!resPatch.ok) throw new Error('Erro ao atualizar participantes ao sair')

      props.sala.participantes = novaQuantidade
      emit('atualizar-participantes', novaQuantidade)
    }

    participando.value = false

  } catch (erro) {
    console.error(erro)
    alert('Não foi possível sair da sala.')
  } finally {
    carregando.value = false
  }
}

onMounted(verificarParticipacao)


watch(() => props.sala?.id || props.sala?.idSala, verificarParticipacao)
</script>

<template>
  <button
    v-if="participando"
    @click="sair"
    :disabled="carregando"
    class="btn-sair"
  >
    {{ carregando ? 'Aguarde...' : 'Sair' }}
  </button>

  <button
    v-else
    @click="entrar"
    :disabled="carregando"
    class="btn-entrar"
  >
    {{ carregando ? 'Aguarde...' : 'Entrar' }}
  </button>
</template>


<style scoped>
.btn-entrar,
.btn-sair {
  font-weight: bold;
  font-size: 0.95rem;
  padding: 5px 15px;
  border-radius: 25px;
  border: none;
  cursor: pointer;
  transition: all 0.2s ease;
  display: inline-flex;
  align-items: center;
  justify-content: center;
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
}

button:hover {
  opacity: 0.9;
  transform: scale(0.95);
}
</style>