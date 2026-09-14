# Atvdd-4-e-5

Atvdd 4

/*
 * Ponteiros para Struct + Vetor de Struct com raylib
 * ---------------------------------------------------------------
 * Evolução da atividade3: agora o foco é em como PONTEIROS PARA
 * STRUCT permitem localizar e alterar um elemento específico dentro
 * de um vetor de struct, sem copiar a struct inteira.
 *
 * O vetor de inimigos é um VETOR DE STRUCT (bloco contíguo, alocado
 * uma única vez com malloc). As funções recebem e retornam
 * "Inimigo *" (ponteiro para struct) para localizar e modificar um
 * elemento específico desse vetor.
 *
 * Conceitos praticados:
 *   - vetor de struct (bloco contíguo de memória)
 *   - ponteiro para struct como retorno de função (localizar um
 *     elemento dentro do vetor e devolver o endereço dele)
 *   - ponteiro para struct como parâmetro de função (alterar o
 *     elemento apontado diretamente, sem cópia)
 *   - enum para representar o estado de cada inimigo
 *   - malloc / free do vetor
 *
 * Compilar (Linux, com raylib instalada):
 *   gcc atividade4.c -o atividade4 -lraylib -lm -lpthread -ldl -lrt -lX11
 */

#include <raylib.h> //
#include <stdlib.h>
#include <time.h>
#include <math.h>

#define LARGURA_JANELA 800
#define ALTURA_JANELA  600
#define RAIO_JOGADOR   20.0f
#define TOTAL_INIMIGOS 8 // 
#define DANO_TIRO      20

typedef enum {
    INIMIGO_VIVO,
    INIMIGO_MORTO, // enum para representar o estado de cada inimigo
} EstadoInimigo;

typedef struct { // struct para representar cada inimigo
    Vector2       pos;
    float         raio;
    int           vida;
    EstadoInimigo estado; 
} Inimigo;

/* preenche o vetor de struct (recebido por ponteiro) com valores iniciais */
void inicializarInimigos(Inimigo *vetor, int n) { //
    for (int i = 0; i < n; i++) {
        Inimigo *ini = (vetor + i); // ponteiro para o i-ésimo elemento
        ini->pos    = (Vector2){ GetRandomValue(30, LARGURA_JANELA - 30),
                                  GetRandomValue(30, ALTURA_JANELA - 30) };
        ini->raio   = 15.0f;
        ini->vida   = 60;
        ini->estado = INIMIGO_VIVO;
    }
}

/* recebe um ponteiro para UM inimigo específico do vetor e altera
 * a vida/estado diretamente na memória original (sem cópia) */
void atingirInimigo(Inimigo *inimigo, int dano) { 
    if (inimigo == NULL || inimigo->estado == INIMIGO_MORTO) return;

    inimigo->vida -= dano;
    if (inimigo->vida <= 0) {
        inimigo->vida = 0;
        inimigo->estado = INIMIGO_MORTO;
    }
}

/* percorre o vetor de struct e RETORNA UM PONTEIRO para o inimigo
 * vivo mais próximo da posição informada (ou NULL se não houver) */
Inimigo *encontrarInimigoMaisProximo(Inimigo *vetor, int n, Vector2 posJogador) {
    Inimigo *maisProximo = NULL;
    float menorDistancia = 0.0f;

    for (int i = 0; i < n; i++) {
        Inimigo *ini = (vetor + i);
        if (ini->estado == INIMIGO_MORTO) continue;

        float dx = ini->pos.x - posJogador.x;
        float dy = ini->pos.y - posJogador.y;
        float distancia = sqrtf(dx * dx + dy * dy);

        if (maisProximo == NULL || distancia < menorDistancia) {
            maisProximo = ini;
            menorDistancia = distancia;
        }
    }
    return maisProximo;
}

void curarTodos(Inimigo *vetor, int n, int cura) { // percorre o vetor de struct e altera a vida de cada inimigo vivo
    for (int i = 0; i < n; i++) { // percorre o vetor de struct
        Inimigo *ini = vetor + i; // ponteiro para o i-ésimo elemento (aritmética de ponteiros)
         if (ini->estado == INIMIGO_VIVO) { // verifica se o inimigo está vivo 
        ini->vida += cura; // altera a vida diretamente na memória original (sem cópia)
            if (ini->vida > 100){ // vida máxima é 100
            ini->vida = 100; // limita a vida máxima 
       }
     }
  }
}
Inimigo *encontrarInimigoMaisFraco(Inimigo *vetor, int n){// percorre o vetor de struct e RETORNA UM PONTEIRO para o inimigo
    // vivo mais fraco (menor vida) encontrado (ou NULL se não houver)
    Inimigo *maisFraco = NULL; // guarda o endereço para o inimigo mais fraco encontrado
    int menorVida = 300; // guarda a menor vida encontrada (inicialmente um valor alto)
    for (int i = 0; i < n; i++) {
        Inimigo *ini = vetor + i; // ponteiro para o i-ésimo elemento
        if (ini->estado == INIMIGO_MORTO) continue; // ignora inimigos mortos
        if (ini->vida < menorVida) { // se a vida do inimigo atual for menor que a menor vida encontrada
            menorVida = ini->vida; // atualiza a menor vida encontrada
            maisFraco = ini; // atualiza o ponteiro para o inimigo mais fraco encontrado
        }
    }
    return maisFraco;   
}
    
void desenharInimigo(Inimigo *ini) {
    if (ini->estado == INIMIGO_MORTO) return;
    Color cor = (ini->vida > 30) ? MAROON : ORANGE;
    DrawCircleV(ini->pos, ini->raio, cor);
    DrawText(TextFormat("%d", ini->vida), ini->pos.x - 8, ini->pos.y - 26, 14, BLACK);
}

int main(void) {
    srand((unsigned int)time(NULL));

    InitWindow(LARGURA_JANELA, ALTURA_JANELA, "Atividade 4 - Ponteiros para Struct + Vetor de Struct");
    SetTargetFPS(60);

    Vector2 jogador = { LARGURA_JANELA / 2.0f, ALTURA_JANELA / 2.0f };

    // vetor de struct: um único bloco contíguo de memória com TOTAL_INIMIGOS structs
    Inimigo *inimigos = (Inimigo *)malloc(TOTAL_INIMIGOS * sizeof(Inimigo));
    inicializarInimigos(inimigos, TOTAL_INIMIGOS);

    while (!WindowShouldClose()) {

        float vel = 250.0f * GetFrameTime();
        if (IsKeyDown(KEY_RIGHT)) jogador.x += vel;
        if (IsKeyDown(KEY_LEFT))  jogador.x -= vel;
        if (IsKeyDown(KEY_UP))    jogador.y -= vel;
        if (IsKeyDown(KEY_DOWN))  jogador.y += vel;

        if (IsKeyPressed(KEY_SPACE)) {
            // ponteiro para o inimigo vivo mais próximo (ou NULL)
            Inimigo *alvo = encontrarInimigoMaisProximo(inimigos, TOTAL_INIMIGOS, jogador);
            atingirInimigo(alvo, DANO_TIRO);
        }
        // cura todos os inimigos vivos em 15 pontos de vida quando a tecla C é pressionada
        if(IsKeyPressed(KEY_C)) {
            curarTodos(inimigos, TOTAL_INIMIGOS,15);
        }
        if(IsKeyPressed(KEY_F)) { // verifica se a tecla F foi pressionada
            Inimigo *alvo = encontrarInimigoMaisFraco(inimigos, TOTAL_INIMIGOS); // ponteiro para o inimigo vivo mais fraco (ou NULL)
            if(alvo != NULL) { // verifica se o ponteiro não é NULL
              atingirInimigo(alvo, DANO_TIRO); // atinge o inimigo mais fraco encontrado
            }
        }
        BeginDrawing();
            ClearBackground(RAYWHITE);

            for (int i = 0; i < TOTAL_INIMIGOS; i++) {
                desenharInimigo(inimigos + i);
            }

            DrawCircleV(jogador, RAIO_JOGADOR, BLUE);

            DrawText("ESPACO atira no inimigo vivo mais proximo", 10, 10, 20, DARKGRAY);
            DrawText("Setas movem o jogador | ESC sai", 10, ALTURA_JANELA - 25, 16, GRAY);

        EndDrawing();
    }

    free(inimigos); // libera o vetor de struct

    CloseWindow();
    return 0;
}


atvdd 5 


/*
 * Vetor de Ponteiros para Struct com raylib
 * ---------------------------------------------------------------
 * Atividade final: reúne todos os conceitos das atividades
 * anteriores em um pequeno "sistema de entidades" (jogador,
 * inimigos e itens).
 *
 * A diferença central para a atividade4 é a forma como a coleção é
 * guardada: em vez de um vetor de struct (bloco contíguo), aqui
 * temos um VETOR DE PONTEIROS PARA STRUCT (Entidade *vetor[N]).
 * Cada posição do vetor guarda apenas um ENDEREÇO; a struct em si
 * fica em um bloco de memória alocado individualmente com malloc.
 * Isso permite, por exemplo, remover uma entidade "no meio" do
 * vetor apenas trocando ponteiros (rápido), sem precisar mover
 * structs inteiras na memória.
 *
 * Conceitos praticados (revisão de todas as atividades):
 *   - ponteiros e aritmética de ponteiros
 *   - alocação dinâmica (malloc/free) de cada struct individual
 *   - struct (Entidade) com campos variados
 *   - enum (TipoEntidade) para diferenciar jogador/inimigo/item
 *   - union (ExtraEntidade) para guardar, no mesmo espaço, o dano
 *     de um inimigo OU o valor de um item, conforme o TipoEntidade
 *   - ponteiro para struct (Entidade *) manipulado por funções
 *   - vetor de ponteiros para struct (Entidade *vetor[N])
 *
 * Compilar (Linux, com raylib instalada):
 *   gcc atividade5.c -o atividade5 -lraylib -lm -lpthread -ldl -lrt -lX11
 */

#include <raylib.h>
#include <stdlib.h>
#include <time.h>
#include <math.h>

#define LARGURA_JANELA  800
#define ALTURA_JANELA   600
#define RAIO_JOGADOR    20.0f
#define MAX_ENTIDADES   30
#define TOTAL_INIMIGOS  5
#define TOTAL_ITENS     6

typedef enum { // enum para diferenciar jogador/inimigo/item
    ENTIDADE_JOGADOR,
    ENTIDADE_INIMIGO,
    ENTIDADE_ITEM
} TipoEntidade;

/* union: só um destes campos faz sentido por vez, dependendo do tipo */
typedef union {
    int dano;   // usado quando tipo == ENTIDADE_INIMIGO
    int valor;  // usado quando tipo == ENTIDADE_ITEM
} ExtraEntidade;

typedef struct { // struct para representar cada entidade (jogador, inimigo ou item)
    TipoEntidade  tipo;
    Vector2       pos;
    float         raio;
    int           vida;
    Color         cor;
    ExtraEntidade extra;
} Entidade;

// vetor de PONTEIROS para struct: cada posição aponta para um bloco
// alocado individualmente com malloc (não é um bloco contíguo único)
Entidade *vetorEntidades[MAX_ENTIDADES];
int totalEntidades = 0;

/* aloca UMA entidade individualmente e devolve o ponteiro para ela */
Entidade *criarEntidade(TipoEntidade tipo, Vector2 pos) {
    Entidade *e = (Entidade *)malloc(sizeof(Entidade));
    if (e == NULL) return NULL;

    e->tipo  = tipo;
    e->pos   = pos;
    e->raio  = (tipo == ENTIDADE_JOGADOR) ? RAIO_JOGADOR
             : (tipo == ENTIDADE_INIMIGO) ? 15.0f : 8.0f;

    switch (tipo) {
        case ENTIDADE_JOGADOR:
            e->vida = 100;
            e->cor  = BLUE;
            break;
        case ENTIDADE_INIMIGO:
            e->vida       = 40;
            e->cor        = MAROON;
            e->extra.dano = GetRandomValue(5, 15);
            break;
        case ENTIDADE_ITEM:
            e->vida        = 1;
            e->cor         = GOLD;
            e->extra.valor = GetRandomValue(5, 20);
            break;
    }
    return e;
}

/* adiciona um ponteiro de entidade no vetor de ponteiros */
void adicionarEntidade(Entidade *e) {
    if (e == NULL || totalEntidades >= MAX_ENTIDADES) return;
    vetorEntidades[totalEntidades] = e;
    totalEntidades++;
}

/* remove a entidade do índice informado: libera a memória dela e
 * substitui a posição vaga pelo ÚLTIMO ponteiro do vetor. Como o
 * vetor guarda apenas ponteiros, isso é apenas uma troca de
 * endereços -- nenhuma struct precisa ser copiada ou movida. */
void removerEntidade(int indice) {
    if (indice < 0 || indice >= totalEntidades) return;

    free(vetorEntidades[indice]);              // libera o bloco alocado
    vetorEntidades[indice] = vetorEntidades[totalEntidades - 1];
    totalEntidades--;
}
 float calcularDistancia(Vector2 a, Vector2 b) { // funçao auxiliar que calcula a distância entre dois pontos (Vector2)
    float dx = a.x - b.x;
    float dy = a.y - b.y;
    return sqrtf(dx * dx + dy * dy);
}
void ordenarPorDistancia(void) { // coloca o inimigo mais próximo do jogador na posição 1 do vetor de ponteiros
    if (totalEntidades < 2) return;

    Entidade *jogador = vetorEntidades[0];
    int indiceMaisProximo = 1;
    
    float menorDistancia = calcularDistancia(jogador->pos, vetorEntidades[1]->pos); // calcula a distância entre o jogador e o primeiro inimigo (posição 1 do vetor)

    for (int i = 2; i < totalEntidades; i++) {
        float dist = calcularDistancia(jogador->pos, vetorEntidades[i]->pos);
        if (dist < menorDistancia) {
            menorDistancia = dist;
            indiceMaisProximo = i;
        }
    }

    if (indiceMaisProximo != 1) {
        Entidade *tmp = vetorEntidades[1];
        vetorEntidades[1] = vetorEntidades[indiceMaisProximo];
        vetorEntidades[indiceMaisProximo] = tmp;
    }
}

bool colidiu(Entidade *a, Entidade *b) {
    float dx = a->pos.x - b->pos.x;
    float dy = a->pos.y - b->pos.y;
    float distancia = sqrtf(dx * dx + dy * dy);
    return distancia <= (a->raio + b->raio);
}

void desenharEntidade(Entidade *e) {
    DrawCircleV(e->pos, e->raio, e->cor);
    if (e->tipo == ENTIDADE_INIMIGO) {
        DrawText(TextFormat("%d", e->vida), e->pos.x - 8, e->pos.y - 26, 14, BLACK);
    }
}

int main(void) {
    srand((unsigned int)time(NULL));

    InitWindow(LARGURA_JANELA, ALTURA_JANELA, "Atividade 5 - Vetor de Ponteiros para Struct");
    SetTargetFPS(60);

    // índice 0 do vetor de ponteiros é sempre o jogador
    Entidade *jogador = criarEntidade(ENTIDADE_JOGADOR,
                                      (Vector2){ LARGURA_JANELA / 2.0f, ALTURA_JANELA / 2.0f });
    adicionarEntidade(jogador);

    for (int i = 0; i < TOTAL_INIMIGOS; i++) {
        Vector2 pos = { GetRandomValue(30, LARGURA_JANELA - 30), GetRandomValue(30, ALTURA_JANELA - 30) };
        adicionarEntidade(criarEntidade(ENTIDADE_INIMIGO, pos));
    }
    for (int i = 0; i < TOTAL_ITENS; i++) {
        Vector2 pos = { GetRandomValue(30, LARGURA_JANELA - 30), GetRandomValue(30, ALTURA_JANELA - 30) };
        adicionarEntidade(criarEntidade(ENTIDADE_ITEM, pos));
    }

    int pontuacao = 0;

    while (!WindowShouldClose()) {

        float vel = 250.0f * GetFrameTime();
        if (IsKeyDown(KEY_RIGHT)) jogador->pos.x += vel;
        if (IsKeyDown(KEY_LEFT))  jogador->pos.x -= vel;
        if (IsKeyDown(KEY_UP))    jogador->pos.y -= vel;
        if (IsKeyDown(KEY_DOWN))  jogador->pos.y += vel;

        // percorre o vetor de ponteiros: cada vetorEntidades[i] já é um "Entidade *"
        for (int i = 1; i < totalEntidades; i++) {
            Entidade *e = vetorEntidades[i];
            if (!colidiu(jogador, e)) continue;

            if (e->tipo == ENTIDADE_ITEM) {
                pontuacao += e->extra.valor;
                removerEntidade(i);
                i--; // a posição i agora tem outra entidade (a que veio do final)
            } else if (e->tipo == ENTIDADE_INIMIGO) {
                jogador->vida -= e->extra.dano;
                if (jogador->vida < 0) jogador->vida = 0;
            }
        }

        if (IsKeyPressed(KEY_SPACE)) {
            // atira no primeiro inimigo vivo encontrado no vetor de ponteiros
            for (int i = 1; i < totalEntidades; i++) {
                Entidade *e = vetorEntidades[i];
                if (e->tipo != ENTIDADE_INIMIGO) continue;
                if (!colidiu(jogador, e) && e->raio > 0) {
                    e->vida -= 20;
                    if (e->vida <= 0) {
                        removerEntidade(i);
                    }
                    break;
                }
            }
        }
        if (IsKeyPressed(KEY_N)) {
             if (totalEntidades < MAX_ENTIDADES) {
        Vector2 posAleatoria = {
            (float)GetRandomValue(50, GetScreenWidth() - 50),
            (float)GetRandomValue(50, GetScreenHeight() - 50)
        };
        Entidade *novoItem = criarEntidade(ENTIDADE_ITEM, posAleatoria);
        if (novoItem != NULL) {
            adicionarEntidade(novoItem);
        }
    }
}
        ordenarPorDistancia(); // coloca o inimigo mais próximo do jogador na posição 1 do vetor de ponteiros

        BeginDrawing();
            ClearBackground(RAYWHITE);

            for (int i = 0; i < totalEntidades; i++) {
                desenharEntidade(vetorEntidades[i]);
            }

            DrawText(TextFormat("Vida: %d   Pontuacao: %d", jogador->vida, pontuacao), 10, 10, 22, DARKGRAY);
            DrawText(TextFormat("Entidades ativas: %d", totalEntidades), 10, 34, 18, GRAY);
            DrawText("Setas movem | ESPACO atira | ESC sai", 10, ALTURA_JANELA - 25, 16, GRAY);

        EndDrawing();
    }

    // libera cada bloco alocado individualmente (cada ponteiro do vetor)
    for (int i = 0; i < totalEntidades; i++) {
        free(vetorEntidades[i]);
    }

    CloseWindow();
    return 0;
}
