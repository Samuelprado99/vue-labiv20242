<template>
  <div class="container">
    <h1>Gerenciar Seções</h1>

    <!-- Formulário -->
    <form @submit.prevent="cadastrarSecao">
      <input v-model="novaSecao.titulo" placeholder="Título" required />
      <textarea v-model="novaSecao.conteudo" placeholder="Conteúdo" required></textarea>
      <input v-model="novaSecao.dataHoraCriacao" type="datetime-local" required />

      <select v-model="novaSecao.trabalhoId" required>
        <option disabled value="">Selecione um trabalho</option>
        <option v-for="t in trabalhos" :key="t.id" :value="t.id">
          {{ t.titulo }}
        </option>
      </select>

      <button type="submit">Cadastrar</button>
    </form>

    <!-- Consulta -->
    <div class="consulta">
      <h3>Consultar Seção</h3>
      <input v-model="filtroTitulo" placeholder="Título da seção" />
      <input v-model="filtroTrabalho" placeholder="Título do trabalho" />
      <button @click="buscarSecao">Buscar</button>
    </div>

    <!-- Lista -->
    <div v-if="secoes.length">
      <table border="1">
        <thead>
          <tr>
            <th>ID</th>
            <th>Título</th>
            <th>Conteúdo</th>
            <th>Antiguidade</th>
            <th>Trabalho</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="s in secoes" :key="s.id">
            <td>{{ s.id }}</td>
            <td>{{ s.titulo }}</td>
            <td>{{ s.conteudo }}</td>
            <td>{{ calcularAntiguidade(s.dataHoraCriacao) }}</td>
            <td>{{ s.trabalho.titulo }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <p v-else>{{ mensagem }}</p>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const secoes = ref([])
const trabalhos = ref([])
const mensagem = ref('')

const novaSecao = ref({
  titulo: '',
  conteudo: '',
  dataHoraCriacao: '',
  trabalhoId: ''
})

const filtroTitulo = ref('')
const filtroTrabalho = ref('')

const baseURL = 'https://verbose-space-system-5gvxwqwrrrg929jg-8080.app.github.dev'


// Buscar todas as seções
const listarSecoes = async () => {
  const resp = await axios.get(`${baseURL}/secao`)
  secoes.value = resp.data
  mensagem.value = secoes.value.length ? '' : 'Nenhuma seção encontrada'
}

// Buscar trabalhos (para o select)
const listarTrabalhos = async () => {
  try {
    const resp = await axios.get(`${baseURL}/trabalho`)
    trabalhos.value = resp.data
  } catch (e) {
    console.log('⚠️ Endpoint /trabalho não encontrado (ignore se não existir)')
  }
}

// Cadastrar nova seção
const cadastrarSecao = async () => {
  if (!novaSecao.value.titulo || !novaSecao.value.conteudo || !novaSecao.value.trabalhoId) {
    alert('Preencha todos os campos obrigatórios.')
    return
  }

  const body = {
    titulo: novaSecao.value.titulo,
    conteudo: novaSecao.value.conteudo,
    dataHoraCriacao: novaSecao.value.dataHoraCriacao,
    trabalho: { id: novaSecao.value.trabalhoId }
  }

  await axios.post(`${baseURL}/secao`, body)
  novaSecao.value = { titulo: '', conteudo: '', dataHoraCriacao: '', trabalhoId: '' }
  listarSecoes()
}

// Buscar seção por filtros
const buscarSecao = async () => {
  const resp = await axios.get(`${baseURL}/secao/busca`, {
    params: {
      secao: filtroTitulo.value,
      trabalho: filtroTrabalho.value
    }
  })
  secoes.value = resp.data
  if (!secoes.value.length) mensagem.value = 'Nenhum registro foi encontrado para os critérios fornecidos'
}

// Calcular antiguidade em meses
const calcularAntiguidade = (data) => {
  if (!data) return '0 meses'
  const criacao = new Date(data)
  const hoje = new Date()
  const meses = (hoje.getFullYear() - criacao.getFullYear()) * 12 + (hoje.getMonth() - criacao.getMonth())
  return `${meses} meses`
}

onMounted(() => {
  listarSecoes()
  listarTrabalhos()
})
</script>

<style scoped>
.container {
  width: 80%;
  margin: auto;
}
form, .consulta {
  margin-bottom: 20px;
}
<style scoped>
.container {
  width: 90%;
  margin: 50px auto;
  background-color: #222;
  color: white;
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 0 20px rgba(0,0,0,0.5);
}

form, .consulta {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  align-items: center;
  margin-bottom: 20px;
}

input, select, textarea, button {
  padding: 8px;
  border-radius: 5px;
  border: none;
}

button {
  background-color: #00C896;
  color: black;
  font-weight: bold;
  cursor: pointer;
}
</style>
