<template>
    <div class="box formulario">
        <div class="columns">
            <div class="column is-5" role="form" aria-label="Formulário para criação de uma nova tarefa">
                <input 
                    type="text" 
                    class="input"
                    placeholder="Qual tarefa você deseja iniciar:"
                    v-model="descricao"
                />
            </div>
            <div class="column is-3">
                <div class="select">
                    <select v-model="idProjeto">
                        <option value="">Selecione o projeto</option>
                        <option 
                            :value="projeto.id"
                            v-for="projeto in projetos"
                            :key="projeto.id"
                        >
                        {{ projeto.nome }}
                        </option>
                    </select>
                </div>
            </div>
            <div class="column">
              <Temporizador @aoTemporizadorFinalizado="finalizarTarefa"/>  
            </div>
        </div>
    </div>
</template>

<script lang="ts">

import { computed, defineComponent } from 'vue';
import { TipoNotificacao } from '@/interfaces/INotificacao'
import { notificacaoMixin } from '@/mixins/notificar'
import Temporizador from './Temporizador.vue'
import { useStore } from 'vuex';
import { key } from '@/store';

export default defineComponent ({
    name: 'Formulário',
    emits: ['aoSalvarTarefa'],
    mixins: [notificacaoMixin],
    data() {
        return {
            descricao: '',
            idProjeto: ''
        }
    },
    components: {
        Temporizador
    },
    methods: {
        finalizarTarefa(tempoDecorrido: number) : void {
            const projeto = this.projetos.find((p) => p.id == this.idProjeto); // primeiro, buscamos pelo projeto
            if(!projeto) { // se o projeto não existe...
                this.notificar(TipoNotificacao.FALHA, 'Ops!', 'Selecione um projeto antes de finalizar a tarefa!')
                return; // ao fazer return aqui, o restante do método salvarTarefa não será executado. chamamos essa técnica de early return :)
            }

            this.$emit('aoSalvarTarefa', {
                duracaoEmSegundos: tempoDecorrido,
                descricao: this.descricao,
                projeto: this.projetos.find(proj => proj.id == this.idProjeto)
            })
            this.descricao = ''
        }
    },
    setup () {
        const store = useStore(key)
        return {
            projetos: computed(() => store.state.projetos),
            store
        }
    }
})

</script>

<style>
.formulario {
    color: var(--texto-primario);
    background-color: var(--bg-primario);
}
</style>