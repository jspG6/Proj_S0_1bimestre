#define _GNU_SOURCE

#include <stdlib.h>
#include <malloc.h>
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>
#define FIBER_STACK 1024*64
#define threads 100



struct c {
 int saldo;
};

typedef struct c conta;
conta c1, c2;
int aux;
int valorAleatorio();
void *RaceCondition(void *arg);

int main(){
	
	void* stack;
	int addCreate;
	int addJoin;
	pthread_t tran[threads];


	stack = malloc( FIBER_STACK );

	if (stack == 0){
		perror("malloc: could not allocate stack");
		exit(1);
	}


	c1.saldo = 100;
	c2.saldo = 0;

	//Inicia as Threads
	for (int i = 0; i < threads; i++){

    //Criação de cada thread com o respectiva transferencia
    addCreate = pthread_create(&tran[i], NULL, RaceCondition, (void*)(intptr_t)i);

	}

  
    for(int i = 0; i < threads; i++){
	  
    //Permite que um aplicativo aguarde o encerramento de um thread. Depois que o thread for encerrado, será salvo na operação
    addJoin = pthread_join(tran[i], NULL);

  }
   printf("\n\n--------------\nSaldo Final c1 (FROM): %d\nSaldo final c2 (TO): %d\n--------------",c1.saldo,c2.saldo);

	free(stack);
		printf("\n\nTransferencias concluidas e memoria liberada.\n");
	return 0;
}

int valorAleatorio(){
    return rand()% 101;
}

void *RaceCondition(void *arg){
	
	
  int i = (intptr_t)arg; 
  int valor = valorAleatorio();

  while (((valor > c1.saldo) && (aux == 0)) || (valor==0)){
    valor = valorAleatorio();
  }

	if ((c1.saldo >= valor) && (aux == 0)){

    printf("\n-- Transferencia (%d) c1 => c2", i + 1);
    printf("\n\nTransferindo R$ %d\n", valor);
    c1.saldo -= valor;
    c2.saldo += valor;

    if (c1.saldo < 0){
      c2.saldo -= valor;
      c1.saldo += valor;
    }

    if (c1.saldo == 0){
      aux = 1;
    }
}

  while (((valor > c2.saldo) && (aux == 1)) || (valor == 0)){
    valor = valorAleatorio();
  }

  if ((c2.saldo >= valor) && (aux == 1)){

    printf("\n-- Transferencia (%d) c2 => c1",i + 1);
    printf("\n\nTransferindo R$ %d\n", valor);
		c2.saldo -= valor;
		c1.saldo += valor;

    if (c2.saldo < 0){
        
      c1.saldo -= valor;
	  c2.saldo += valor;
    }

    if(c2.saldo == 0){
      aux =0;
    }
}

 printf("Transferencia %d concluída com sucesso!\n", i + 1);
 printf("Saldo de c1: %d\n", c1.saldo);
 printf("Saldo de c2: %d\n", c2.saldo);
 
 return 0;   
}

