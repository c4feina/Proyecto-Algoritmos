#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <time.h>
#include <ctype.h>

#define MAX_JUGADORES 8
#define LONG_NOMBRE 50
#define STAT_BASE 11
#define MAX_HABILIDADES 3

typedef struct {
    int caras;
    int cantidad;
} Dado;

// enumeraciones para las razas, tipos de elfos, géneros y estirpes
typedef enum { HUMANO = 1, ELFO, ORCO, ENANO, MUERTO } Raza;
typedef enum { QUEL = 1, BOS, DUN } TipoElfo;
typedef enum { MASCULINO = 1, FEMENINO } Genero;
typedef enum { HUMANA = 1, ELFICA, ORCA } Estirpe;

typedef struct {
    char nombre[30];
    int dano_base;
    int costo_energia;
} Habilidad;

typedef struct {
    char nombre[LONG_NOMBRE];
    Raza raza;
    Genero genero;
    Estirpe estirpe;
    TipoElfo tipo_elfo;
    int muerto_viviente;
    
    int fuerza;
    int agilidad;
    int vitalidad;
    int inteligencia;
    int destreza;
    int suerte;
    int energia;
    
    Habilidad habilidades[MAX_HABILIDADES];
    int nivel;
    int experiencia;
} Personaje;

typedef struct {
    int total;
    int por_raza[5];
    int por_genero[2];
    int muertos;
    int puntos_extra;
    int combates_ganados;
} Estadisticas;

// prototipos de funciones
void mostrarEstadisticas(Estadisticas stats);
void inicializarPersonaje(Personaje *p);
void mostrarMenuPrincipal();
void cargarDatosPersonaje(Personaje *p, Estadisticas *stats);
void aplicarModificadores(Personaje *p, Estadisticas *stats);
void mostrarPersonaje(Personaje p);
void mostrarEstadisticas(Estadisticas stats);
void calcularEstadisticasJefe(Personaje equipo[], int cantidad);
void combate(Personaje *atacante, Personaje *defensor, Estadisticas *stats);
int lanzarDados(int cantidad, int caras);
void subirNivel(Personaje *p);
void menuCombate(Personaje equipo[], int cantidad, Estadisticas *stats);

int main() {
    srand(time(NULL));
    Personaje equipo[MAX_JUGADORES];
    Estadisticas stats = {0};
    char opcion;
    int cantidad = 0;
    
    printf("=== SISTEMA DE CREACION DE PERSONAJES ===");
    
    do {
        if (cantidad >= MAX_JUGADORES) {
            printf("\n¡Maximo de %d jugadores alcanzado!\n", MAX_JUGADORES);
            break;
        }
        
        printf("\n\n--- Personaje %d ---\n", cantidad + 1);
        Personaje nuevo;
        inicializarPersonaje(&nuevo);
        cargarDatosPersonaje(&nuevo, &stats);
        aplicarModificadores(&nuevo, &stats);
        equipo[cantidad++] = nuevo;
        
        mostrarPersonaje(nuevo);
        
        if (cantidad < MAX_JUGADORES) {
            printf("\n¿Crear otro personaje? (s/n): ");
            scanf(" %c", &opcion);
        }
    } while ((opcion == 's' || opcion == 'S') && cantidad < MAX_JUGADORES);
    
    menuCombate(equipo, cantidad, &stats);
    
    printf("\n\n=== RESUMEN FINAL ===");
    mostrarEstadisticas(stats);
    
    if (cantidad > 1) {
        calcularEstadisticasJefe(equipo, cantidad);
    }
    
    printf("\n\nPrograma terminado. Presione Enter para salir...");
    getchar(); getchar();
    return 0;
}

void inicializarPersonaje(Personaje *p) {
    p->fuerza = p->agilidad = p->vitalidad = STAT_BASE;
    p->inteligencia = p->destreza = p->suerte = STAT_BASE;
    p->energia = 100;
    p->muerto_viviente = 0;
    p->nivel = 1;
    p->experiencia = 0;
    
    // Habilidades básicas
    strcpy(p->habilidades[0].nombre, "Golpe basico");
    p->habilidades[0].dano_base = 10;
    p->habilidades[0].costo_energia = 0;
    
    strcpy(p->habilidades[1].nombre, "Defensa total");
    p->habilidades[1].dano_base = 5;
    p->habilidades[1].costo_energia = 20;
}

void cargarDatosPersonaje(Personaje *p, Estadisticas *stats) {
    int opcion;
    
    printf("\nNombre del personaje: ");
    scanf("%s", p->nombre);
    
    printf("\nRazas:\n1.Humano 2.Elfo 3.Orco 4.Enano 5.Muerto\nSeleccione: ");
    scanf("%d", &opcion);
    p->raza = (opcion == 5) ? MUERTO : (Raza)(opcion - 1);
    
    if(p->raza == ELFO) {
        printf("Tipos Elfo:\n1.Quel 2.Bos 3.Dun\nSeleccione: ");
        scanf("%d", &opcion);
        p->tipo_elfo = opcion-1;
        sprintf(p->nombre, "%s %s", (opcion==1)?"Quel":(opcion==2)?"Bos":"Dun", p->nombre);
    }
    
    printf("\nGenero:\n1.Masculino 2.Femenino\nSeleccione: ");
    scanf("%d", &opcion);
    p->genero = (opcion == 1) ? MASCULINO : FEMENINO;
    
    printf("\nEstirpe:\n1.Humana 2.Elfica 3.Orca\nSeleccione: ");
    scanf("%d", &opcion);
    p->estirpe = (Estirpe)opcion;
    
    if(p->raza == MUERTO) {
        p->muerto_viviente = 1;
        stats->muertos++;
    }
}

void aplicarModificadores(Personaje *p, Estadisticas *stats) {
    stats->total++;
    stats->por_raza[p->raza]++;
    stats->por_genero[p->genero]++;
    
    if(p->genero == FEMENINO) {
        p->agilidad += 2;
        p->inteligencia += 1;
        stats->puntos_extra += 3;
    } else {
        p->fuerza += 2;
        p->vitalidad += 1;
        stats->puntos_extra += 3;
    }
    
    switch(p->raza) {
        case HUMANO:
            p->destreza += 2;
            p->suerte += 1;
            stats->puntos_extra += 3;
            break;
        case ELFO:
            if(p->tipo_elfo == QUEL) {
                p->inteligencia += 3;
                p->destreza += 1;
            } else if(p->tipo_elfo == BOS) {
                p->agilidad += 3;
                p->destreza += 2;
            } else if(p->tipo_elfo == DUN) {
                p->fuerza += 2;
                p->suerte += 1;
            }
            stats->puntos_extra += 5;
            break;
        case ORCO:
            p->fuerza += 4;
            p->vitalidad += 2;
            stats->puntos_extra += 6;
            break;
        case ENANO:
            p->vitalidad += 3;
            p->fuerza += 2;
            stats->puntos_extra += 5;
            break;
        case MUERTO:
            p->inteligencia += 5;
            p->suerte -= 3;
            stats->puntos_extra += 2;
            break;
        default:
            printf("\nError: Raza no válida.\n");
            break;
    }
    
    switch(p->estirpe) {
        case ELFICA:
            p->agilidad += 2;
            p->energia += 20;
            break;
        case ORCA:
            p->fuerza += 3;
            p->energia -= 10;
            break;
        default:
            printf("\nError: Estirpe no válida.\n");
            break;
    }
}

void combate(Personaje *atacante, Personaje *defensor, Estadisticas *stats) {
    int turno = 1;
    Dado ataque = {6, 2};
    Dado defensa = {4, 1};
    
    printf("\n\n=== COMBATE INICIADO ===");
    printf("\n%s vs %s", atacante->nombre, defensor->nombre);
    
    while(atacante->vitalidad > 0 && defensor->vitalidad > 0) {
        printf("\n\n--- Turno %d ---", turno++);
        
        // Turno del atacante
        int atk = lanzarDados(ataque.cantidad, ataque.caras) + atacante->fuerza;
        int def = lanzarDados(defensa.cantidad, defensa.caras) + defensor->agilidad;
        
        printf("\n%s ataca con %d vs %s defiende con %d", 
              atacante->nombre, atk, defensor->nombre, def);
        
        if(atk > def) {
            int dano = atk - def;
            defensor->vitalidad -= dano;
            printf("\n¡%s recibe %d puntos de daño!", defensor->nombre, dano);
        } else {
            printf("\n¡%s bloquea el ataque!", defensor->nombre);
        }
        
        // Turno del defensor
        if(defensor->vitalidad > 0) {
            atk = lanzarDados(ataque.cantidad, ataque.caras) + defensor->fuerza;
            def = lanzarDados(defensa.cantidad, defensa.caras) + atacante->agilidad;
            
            printf("\n%s contraataca con %d vs %s defiende con %d", 
                  defensor->nombre, atk, atacante->nombre, def);
            
            if(atk > def) {
                int dano = atk - def;
                atacante->vitalidad -= dano;
                printf("\n¡%s recibe %d puntos de daño!", atacante->nombre, dano);
            } else {
                printf("\n¡%s bloquea el contraataque!", atacante->nombre);
            }
        }
        
        printf("\nVida restante:");
        printf("\n%s: %d", atacante->nombre, atacante->vitalidad);
        printf("\n%s: %d", defensor->nombre, defensor->vitalidad);
    }
    
    Personaje *ganador = (atacante->vitalidad > 0) ? atacante : defensor;
    printf("\n\n¡%s ha ganado el combate!", ganador->nombre);
    stats->combates_ganados++;
    ganador->experiencia += 50;
    
    if(ganador->experiencia >= 100) {
        subirNivel(ganador);
    }
}

void subirNivel(Personaje *p) {
    p->nivel++;
    p->experiencia = 0;
    p->fuerza += 2;
    p->agilidad += 2;
    p->vitalidad += 5;
    printf("\n¡%s ha subido al nivel %d!", p->nombre, p->nivel);
}

void menuCombate(Personaje equipo[], int cantidad, Estadisticas *stats) {
    int opcion;
    do {
        printf("\n\n=== MENU PRINCIPAL ===");
        printf("\n1. Iniciar combate");
        printf("\n2. Mostrar personajes");
        printf("\n3. Estadisticas");
        printf("\n4. Salir");
        printf("\nSeleccione opcion: ");
        scanf("%d", &opcion);
        
        switch(opcion) {
            case 1:
                if(cantidad >= 2) {
                    int p1, p2;
                    printf("\nSeleccione dos personajes (1-%d): ", cantidad);
                    scanf("%d %d", &p1, &p2);
                    if(p1 >= 1 && p1 <= cantidad && p2 >= 1 && p2 <= cantidad && p1 != p2) {
                        combate(&equipo[p1-1], &equipo[p2-1], stats);
                    }
                } else {
                    printf("\n¡Necesitas al menos 2 personajes!");
                }
                break;
            case 2:
                for(int i = 0; i < cantidad; i++) {
                    mostrarPersonaje(equipo[i]);
                }
                break;
            case 3:
                mostrarEstadisticas(*stats);
                break;
        }
        
        if(opcion != 4) {
            printf("\nPresione Enter para continuar...");
            getchar(); getchar();
        }
    } while(opcion != 4);
}

// Resto de funciones de mostrar estadísticas y cálculos...
