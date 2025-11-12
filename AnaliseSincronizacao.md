# 🧩 Análise de Sincronização em Programas Java Multithread

---

## 📚 Sumário
1. [Introdução](#introdução)  
2. [Analisando o Problema de Sincronização](#analisando-o-problema-de-sincronização)  
3. [Entendendo o Problema](#entendendo-o-problema)  
4. [Por que a Sincronização é Importante?](#por-que-a-sincronização-é-importante)  
5. [Atividade Prática 01 – Sem Sincronização (`MeuDadoThreads`)](#atividade-prática-01--sem-sincronização-meudadothreads)  
6. [Atividade Prática 02 – Sincronização com Monitores (`MeuDadoMonitorJava`)](#atividade-prática-02--sincronização-com-monitores-meudadomonitorjava)  
7. [Atividade Prática 03 – Sincronização com Eventos (`MeuDadoEventJava`)](#atividade-prática-03--sincronização-com-eventos-meudadoeventjava)  
8. [Comparativo entre as Abordagens](#comparativo-entre-as-abordagens)  
9. [Conclusão e Opinião Técnica](#conclusão-e-opinião-técnica)  

---

## 🏁 Introdução

Este relatório apresenta uma análise técnica detalhada de três implementações do mesmo problema de **produção e consumo de dados** usando **threads em Java**.  
Cada versão aplica um mecanismo diferente de controle de concorrência:

- 🟥 **Atividade 01:** sem sincronização  
- 🟨 **Atividade 02:** sincronização com **monitores**  
- 🟩 **Atividade 03:** sincronização com **eventos (`wait` e `notify`)**

O objetivo é compreender como a presença ou ausência de sincronização afeta a **integridade dos dados**, a **ordem de execução** e o **comportamento geral** do sistema.

---

## ⚙️ Analisando o Problema de Sincronização

O principal problema no código inicial (`MeuDadoThreads.java`) é a **ausência de sincronização** ao acessar o recurso compartilhado — a variável `Dado`.  
Tanto o produtor quanto o consumidor acessam e modificam essa variável simultaneamente, sem qualquer proteção contra condições de corrida (*race conditions*).

---

## 🧠 Entendendo o Problema

- **Recurso Compartilhado:** a variável `Dado` é usada por duas threads simultaneamente.  
- **Acesso Concorrente:** ambas podem ler e escrever na mesma variável ao mesmo tempo.  
- **Condição de Corrida:** o valor final de `Dado` é imprevisível, dependendo da ordem de execução.  
- **Risco:** o consumidor pode ler valores incorretos, repetidos ou desatualizados.

💡 *Exemplo:*  
Se o produtor está armazenando `10` e o consumidor lê nesse instante, o valor pode ser o antigo (`9`) ou até `0`, dependendo do ponto de interrupção do processador.

---

## 🛡️ Por que a Sincronização é Importante?

| 💡 Benefício | 📖 Descrição |
|--------------|--------------|
| **Integridade dos dados** | Garante que as alterações sejam consistentes e corretas. |
| **Consistência** | Mantém a ordem lógica e previsível das operações. |
| **Prevenção de Corridas** | Evita resultados aleatórios devido a acessos simultâneos. |
| **Previsibilidade** | Torna o comportamento do programa determinístico. |

---

## 🧩 Atividade Prática 01 – Sem Sincronização (`MeuDadoThreads`)

### 🔍 Código Principal
```java
class MeuDadoThreads {
    private int Dado;
    public void armazenar(int Dado) { this.Dado = Dado; }
    public int carregar() { return this.Dado; }
}

class ProdutorThreads implements Runnable {
    MeuDadoThreads dado;
    public ProdutorThreads(MeuDadoThreads dado) { this.dado = dado; }
    public void run() {
        for (int i = 0; i < 30; i++) {
            dado.armazenar(i);
            System.out.println("Produtor: " + i);
            try { Thread.sleep((int)(Math.random() * 500)); } catch (InterruptedException e) {}
        }
    }
}

class ConsumidorThreads implements Runnable {
    MeuDadoThreads dado;
    public ConsumidorThreads(MeuDadoThreads dado) { this.dado = dado; }
    public void run() {
        for (int i = 0; i < 30; i++) {
            System.out.println("Consumidor: " + dado.carregar());
            try { Thread.sleep((int)(Math.random() * 500)); } catch (InterruptedException e) {}
        }
    }
}

class MeuDadoThreadsJava {
    public static void main(String[] args) {
        MeuDadoThreads dado = new MeuDadoThreads();
        new Thread(new ProdutorThreads(dado)).start();
        new Thread(new ConsumidorThreads(dado)).start();
    }
}
Produtor: 0
Consumidor: 0
Produtor: 1
Consumidor: 1
Produtor: 3
Consumidor: 2
Produtor: 4
Consumidor: 5
...
class MeuDadoMonitor {
    private int Dado;
    private boolean Pronto;
    private boolean Ocupado;

    public MeuDadoMonitor() {
        Pronto = false;
        Ocupado = true;
    }

    public void armazenar(int Dado) {
        while (!Ocupado);
        synchronized (this) {
            this.Dado = Dado;
            Ocupado = false;
            Pronto = true;
        }
    }

    public int carregar() {
        while (!Pronto);
        synchronized (this) {
            Pronto = false;
            Ocupado = true;
            return this.Dado;
        }
    }
}
Armazenar Iniciando...
Armazenar Finalizando...
Produtor usando Monitor: 0
Carregar Iniciando...
Carregar Finalizando...
Consumidor usando Monitor: 0
class MeuDadoEvent {
    private int Dado;
    private boolean Pronto;

    public MeuDadoEvent() { Pronto = false; }

    public synchronized void armazenar(int Data) {
        while (Pronto)
            try { wait(); } catch (InterruptedException e) {}
        this.Dado = Data;
        Pronto = true;
        notify();
    }

    public synchronized int carregar() {
        while (!Pronto)
            try { wait(); } catch (InterruptedException e) {}
        Pronto = false;
        notify();
        return this.Dado;
    }
}
Produtor usando Eventos: 0
Consumidor usando Eventos: 0
Produtor usando Eventos: 1
Consumidor usando Eventos: 1
...
