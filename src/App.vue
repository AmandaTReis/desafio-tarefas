<template>
  <div id="app">
    <div class="panel panel-primary">
      <div class="panel-heading">
          <div class="row ">
              <h2>Lista de Tarefas</h2>
          </div>      
          <div class="row alinhar-direita">
               <i class="fa fa-filter fa-border fa-lg" data-toggle="tooltip" data-placement="top" title="Filtrar" @click="toggleFiltro('data')"></i>
          </div>
          <div v-show="filtro.ativo" > 
            <div class="row">
                <div class="col-md-12">
                   <div class="input-group">
                    <input  v-model="filtro.textoPesquisa" type="text" class="form-control" name="pesquisa" placeholder="Pesquisar tarefa...">
                    <span class="input-group-addon"><i class="fa fa-search" aria-hidden="true"></i></span>            
                  </div>                 
                </div>                
            </div>
            <div class='row botoes-ordenacao'>
                  <div class="col-md-12">
                      <button type="button" class="btn btn-default" @click="ordena('data')">Data <i class="fa fa-arrow-down" aria-hidden="true"></i></button>
                      <button type="button" class="btn btn-default" @click="ordena('ultimaModificacao')">Última Modificação <i class="fa fa-arrow-down" aria-hidden="true"></i></button>
                      <button type="button" class="btn btn-default" @click="ordena('dataFinalizada')">Finalizado em <i class="fa fa-arrow-down" aria-hidden="true"></i></button>
                  </div>
            </div>
          </div>            
      </div>  
      <div class="panel-body">

          <input class="form-control" v-model="tarefa.titulo" type="text" placeholder="Lembrar de..."/>
          <input class="form-control" v-model="tarefa.descricao" type="text" placeholder="Descrição..."/>
          <input class="form-control" v-model="tarefa.data" type="date" />
          <button class="btn btn-primary btn-block" :disabled="!tarefa.titulo && !tarefa.descricao" @click="adicionarTarefa()">Adicionar Tarefa</button> 
          <div class="listaTarefas">     
              <div class="row tarefa" v-for="tarefa in tarefas | filterBy filtro.textoPesquisa in 'titulo' 'descricao' | orderBy filtro.ordenacao">
                 <div class="row">
                    <div class="col-md-8">
                        <h4 v-bind:class="{ finalizada: tarefa.finalizada }">{{tarefa.titulo}}</h4>
                    </div>
                    <div class="col-md-4 icones">
                        <i class="fa fa-trash fa-lg" aria-hidden="true" data-toggle="tooltip" data-placement="top" title="Deletar Tarefa"  @click="deletarTarefa(tarefa)" ></i>
                        <i class="fa fa-pencil fa-lg" aria-hidden="true" data-toggle="tooltip" data-placement="top" title="Editar Tarefa" @click="editarTarefa(tarefa)"></i>
                        <i class="fa fa-check fa-lg" aria-hidden="true" data-toggle="tooltip" data-placement="top" title="Concluir Tarefa"  @click="finalizarTarefa(tarefa)" ></i>
                    </div>
                 </div>
                 <div class="row" >
                    <div class="col-md-12 descricao" ><p v-bind:class="{ finalizada: tarefa.finalizada }">{{ tarefa.descricao }}</p></div>
                 </div>
                 <div class="row" >
                        <div class="col-md-4">
                            <div class="row"><div class="col-md-12 descricao" ><small><strong>Data:</strong></small></div></div>  
                            <div class="row"><div class="col-md-12 descricao" ><small>{{ tarefa.data |dataSimples}}</small></div></div>                     
                        </div>
                        <div class="col-md-4">
                            <div class="row"><div class="col-md-12 descricao" ><small><strong>Última modificação:</strong></small></div></div>
                            <div class="row"><div class="col-md-12 descricao" ><small> {{ tarefa.ultimaModificacao |dataHora}}</small></div></div>
                        </div>
                        <div class="col-md-4" v-show="tarefa.finalizada">
                            <div class="row"><small><div class="col-md-12 descricao" ><strong>Finalizada em:</strong></small></div></div>
                            <div class="row"><small><div class="col-md-12 descricao" >{{ tarefa.dataFinalizada | dataHora}}</small></div></div>
                        </div>                                        
                 </div>                               
              </div>
          </div> 
      </div>
    </div>
  </div>
</template>

<script>
var moment = require('moment');
var tarefasArmazenadas = [];
if(localStorage.tarefas){
   tarefasArmazenadas = JSON.parse(localStorage.tarefas);
}
export default {
  data () {
    return {
      tarefas: tarefasArmazenadas,
      filtro:{
        ativo:'false',
        textoPesquisa:'',
        ordenacao:'data'

      },
      tarefa:{
        titulo:'',
        descricao:'',
        data:'',
        ultimaModificacao:'',
        finalizada:false,
        dataFinalizada:''

      }
    }
  },
  watch: {
    tarefas: {
      handler: function (todos) {
        this.armazenaTarefa();
      },
      deep: true
    }
  },
  methods:{
    adicionarTarefa() {
      this.tarefa.ultimaModificacao = moment();
      this.tarefas.push(this.tarefa);
      this.tarefa = {
        titulo:'',
        descricao:'',
        data:'',
        ultimaModificacao:'',
        finalizada:false,
        dataFinalizada:''

      };
    },
    deletarTarefa(tarefa) {    
      this.tarefas.$remove(tarefa);
    },
    editarTarefa(tarefa) {    
       this.deletarTarefa(tarefa);
       this.tarefa = tarefa;
       this.$els.tarefa.focus();
    },
    finalizarTarefa(tarefa) { 
        tarefa.ultimaModificacao = moment();
        if(tarefa.finalizada){
          tarefa.finalizada = false;
          tarefa.dataFinalizada = '';

        }else{
          tarefa.finalizada=true;
          tarefa.dataFinalizada = tarefa.ultimaModificacao;
        }   
        
    },    
    armazenaTarefa(){
      localStorage.setItem("tarefas", JSON.stringify(this.tarefas));

    },
    ordena(orderby){
      this.filtro.ordenacao=orderby;
    },
    toggleFiltro(){
      this.filtro.textoPesquisa = '';
      if(this.filtro.ativo){
        this.filtro.ativo = false;
      }else{
        this.filtro.ativo = true;
      }
    }

  },
  filters: {
    dataHora: function (date) {
      if(date != ''){return moment(date).format('DD/MM/YYYY h:mm:ss')};
    },
    dataSimples: function (date) {
      if(date != ''){return moment(date).format('DD/MM/YYYY')};
    }    
  }
}

</script>

<style>

</style>
