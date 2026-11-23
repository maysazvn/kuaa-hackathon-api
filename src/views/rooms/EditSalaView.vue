<script setup>
import { ref, watch, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute()
const router = useRouter()
const nome = ref('')
const descricao = ref('')
const banner = ref('')
const sala = ref(null)

// Carrega os dados atuais da sala da API
onMounted(async () => {
    try {
        const resposta = await fetch(`http://localhost:3000/salas/${route.params.id}`)
        if (resposta.ok) {
            sala.value = await resposta.json()
            nome.value = sala.value.nome
            descricao.value = sala.value.desc
            banner.value = sala.value.banner
        }
    } catch (error) {
        console.error('Erro ao buscar dados para edição:', error)
    }
})

async function edit() {
    const dadosAtualizados = {
        ...sala.value,
        nome: nome.value,
        desc: descricao.value,
        banner: banner.value
    }

    try {
        // Envia a requisição PUT para atualizar o recurso completo
        const resposta = await fetch(`http://localhost:3000/salas/${route.params.id}`, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(dadosAtualizados)
        })

        if (resposta.ok) {
            router.push(`/salas/${route.params.id}`)
        } else {
            alert('Erro ao atualizar a sala.')
        }
    } catch (error) {
        console.error('Erro na atualização:', error)
    }
}

function cancelar() {
    router.push(`/salas/${route.params.id}`)
}
</script>

<template>
    <div class="container-edit">
        <div class="header-pagina">
            <div class="titulo">
                <RouterLink :to="`/salas/${route.params.id}`" class="voltar"> 
                    <font-awesome-icon icon="chevron-left"></font-awesome-icon> Editar sala
                </RouterLink>
            </div>
        </div>

        <div class="card-editar">
            <div class="conteudo-editar">
                <div class="area-imagem">
                    <img :src="banner || 'https://i.pinimg.com/736x/ed/b6/06/edb606bf10aad2fdfd3a854758ee0042.jpg'" class="foto-preview" alt="Preview da imagem" />
                    <div class="input-imagem-container">
                        <input type="text" v-model="banner" placeholder="URL da Imagem" class="input-url" />
                    </div>
                </div>

                <div class="formulario">
                    <div class="campo">
                        <label>Nome da sala</label>
                        <div class="input-box">
                            <input type="text" v-model="nome" maxlength="25" placeholder="Psikolera" />
                            <span class="contador">{{ nome.length }}/25</span>
                        </div>
                    </div>

                    <div class="campo">
                        <label>Descrição</label>
                        <div class="input-box">
                            <textarea v-model="descricao" maxlength="300" rows="4" placeholder="Descrição da sala..."></textarea>
                            <span class="contador">{{ descricao.length }}/300</span>
                        </div>
                    </div>
                </div>
            </div>

            <div class="botoes-acoes">
                <button @click="edit" class="btn-confirmar">Confirmar</button>
                <button @click="cancelar" class="btn-cancelar">Cancelar</button>
            </div>
        </div>
    </div>
</template>

<style scoped>

.container-edit {
  max-width: 850px;
  margin: 0 auto;
  padding: 20px;
  color: #d9d9d9;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.header-pagina {
  margin-bottom: 20px;
  width: 100%;
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

.card-editar {
  background-color: #242424;
  border-radius: 20px;
  padding: 35px;
  display: flex;
  flex-direction: column;
  gap: 30px;
  box-shadow: 0 0 15px #00000049;
  width: 100%;
}

.conteudo-editar {
  display: flex;
  gap: 30px;
  width: 100%;
}

.area-imagem {
  width: 250px;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.foto-preview {
  width: 100%;
  height: 200px;
  object-fit: cover;
  border-radius: 18px;
  background-color: #ffffff;
  position: relative;
}

.input-url {
  border: 2px solid #3c3c3c;
  border-radius: 10px;
  padding: 6px 10px;
  color: #8f8f8f;
  width: 100%;
  max-width: 500px;
  resize: none;
  outline: none;
}

.input-url:focus {
  border-color: #f8d668c5;
}

.formulario {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.campo {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.campo label {
  font-size: 1.1rem;
  font-weight: bold;
  color: #d9d9d9;
}

.input-box {
  position: relative;
  display: flex;
  align-items: center;
}

.input-box input,
.input-box textarea {
  border: 2px solid #3c3c3c;
  border-radius: 10px;
  padding: 6px 10px;
  color: #8f8f8f;
  width: 100%;
  max-width: 500px;
  resize: none;
  outline: none;
}

.input-box textarea {
  height: 90px;
  width: 300px;
}

.input-box input:focus,
.input-box textarea:focus {
  border-color: #f8d668c5;
}

.contador {
  color: #8f8f8f;
  font-size: 0.8rem;
  position: absolute;
  right: 10px;
  bottom: 10px;
}

.botoes-acoes {
  display: flex;
  justify-content: center;
  gap: 30px;
  margin-top: 30px;
}

.btn-confirmar,
.btn-cancelar {
  background-color: #f4d068;
  color: #1e1e1e;
  padding: 5px 40px;
  border-radius: 25px;
  font-size: 1rem;
  font-weight: bold;
  cursor: pointer;
}

.btn-confirmar:hover,
.btn-cancelar:hover {
  opacity: 0.9;
  transform: scale(0.97);
  transition: 0.3s;
}

.btn-cancelar {
  background: #848484;
}

@media (max-width: 768px) {
  .container-edit{
    padding: 0;

  }

  .card-editar{
    display: flex;
    flex-direction: column;
  }

  .conteudo-editar{
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  .img,
  .area-imagem {
    width: 100%;
    max-width: 300px;
  }

  .formulario{
    width: 100%;
  }

  input .input-url{
    display: flex;
    flex-direction: column;
    justify-content: center;
    width: 100%;
  }

  .input input,
.input textarea {
  width: 100%;
}
}
</style>
