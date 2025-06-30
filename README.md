#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

// --- Estructuras ---
// Estructura de una tarea
typedef struct {
    char titulo[101];    // Ahora hasta 100 caracteres + 1 para el '\0'
    char descripcion[501]; // Ahora hasta 500 caracteres + 1 para el '\0'
    char estado[15];      // Ej: "Pendiente", "En curso", "Terminada", "Cancelada"
    char dificultad[4];   // Ej: "+--", "++-", "+++"
    char vencimiento[11]; // Ej: "DD/MM/AAAA"
    char creacion[11];    // Ej: "DD/MM/AAAA"
} Tarea;

// --- Constantes y Variables Globales ---
#define MAX_TAREAS 10 // Define el número máximo de tareas que se pueden almacenar
Tarea tareas[MAX_TAREAS]; // Array para almacenar las tareas
int totalTareas = 0; // Contador de tareas actuales, inicializado en 1 por la tarea de ejemplo

// --- Prototipos de Funciones ---
void limpiarPantalla();
void mostrarMenuPrincipal(char nombre[]);
void mostrarSubmenuFiltrarTareas();
void buscarTareaPorTitulo();
void agregarTareaNueva();
void verTareasFiltradas(const char *filtro);
void crearTarea(); // Función para la lógica de creación de una tarea
void mostrarDetalleTarea(Tarea *t);
void editarTarea(Tarea *t);
void listarTodasLasTareas(); // Nueva función para listar todas las tareas sin filtro
void eliminarTarea(int indice); // Nueva función para eliminar una tarea

// --- MAIN PRINCIPAL ---
int main() {
    char nombre[50] = "Olivia"; // Nombre del usuario
    int opcion;

    // Bucle principal del programa
    while (1) {
        limpiarPantalla(); // Limpia la consola antes de mostrar el menú
        mostrarMenuPrincipal(nombre); // Muestra el menú principal

        printf("> ");
        // Lee la opción del usuario. scanf puede dejar un '\n' en el buffer,
        // por eso se usa getchar() después para limpiarlo.
        if (scanf("%d", &opcion) != 1) { // Verifica que la entrada sea un número
            printf("Entrada no válida. Por favor, ingresa un número.\n");
            // Limpiar el buffer de entrada en caso de entrada no numérica
            while (getchar() != '\n'); 
            printf("\nPresiona ENTER para continuar...");
            getchar();
            continue; // Vuelve al inicio del bucle
        }
        getchar(); // Limpia el buffer de entrada

        // Manejo de las opciones del menú principal
        switch (opcion) {
            case 1: mostrarSubmenuFiltrarTareas(); break;
            case 2: buscarTareaPorTitulo(); break;
            case 3: agregarTareaNueva(); break; 
            case 0: 
                printf("¡Hasta luego!\n");
                exit(0); // Termina el programa
            default:
                printf("Opción no válida. Por favor, selecciona una de las opciones del menú.\n");
                break;
        }

        printf("\nPresiona ENTER para continuar...");
        getchar(); // Espera a que el usuario presione ENTER para continuar
    }

    return 0; // Indica que el programa finalizó correctamente
}


// Muestra el menú principal de la aplicación.
void mostrarMenuPrincipal(char nombre[]) {
    printf("¡Hola %s!\n\n", nombre);
    printf("¿Qué deseas hacer?\n\n");
    printf("[1] Ver Mis Tareas.\n");
    printf("[2] Buscar una Tarea.\n");
    printf("[3] Agregar una Tarea.\n"); 
    printf("[0] Salir.\n\n");
}

// Muestra el submenú para filtrar y ver tareas.
void mostrarSubmenuFiltrarTareas() {
    int opcion;
    limpiarPantalla();
    printf("¿Qué tareas deseas ver?\n\n");
    printf("[1] Todas\n");
    printf("[2] Pendientes\n");
    printf("[3] En curso\n");
    printf("[4] Terminadas\n");
    printf("[0] Volver\n\n");
    printf("> ");
    if (scanf("%d", &opcion) != 1) {
        printf("Entrada no válida. Volviendo al menú principal.\n");
        while (getchar() != '\n');
        return;
    }
    getchar(); // Limpia el buffer

    switch(opcion) {
        case 1: listarTodasLasTareas(); break; // Llama a la nueva función
        case 2: verTareasFiltradas("Pendiente"); break;
        case 3: verTareasFiltradas("En curso"); break;
        case 4: verTareasFiltradas("Terminada"); break;
        case 0: return; // Vuelve al menú principal
        default: printf("Opción no válida.\n");
    }
}

// Muestra todas las tareas sin aplicar ningún filtro.
void listarTodasLasTareas() {
    limpiarPantalla();
    printf("--- Todas tus tareas ---\n\n");
    if (totalTareas == 0) {
        printf("No tienes tareas registradas.\n");
        return;
    }
    for (int i = 0; i < totalTareas; i++) {
        printf("[%d] Título: %s | Estado: %s | Vencimiento: %s\n", 
               i + 1, tareas[i].titulo, tareas[i].estado, tareas[i].vencimiento);
    }

    // --- NUEVA FUNCIONALIDAD: Permitir seleccionar tarea para ver/editar ---
    printf("\n¿Deseas ver los detalles o editar alguna tarea?\nIntroduce el número de la tarea o 0 para volver.\n> ");
    int seleccion;
    if (scanf("%d", &seleccion) != 1) {
        printf("Entrada no válida. Volviendo al menú anterior.\n");
        while (getchar() != '\n');
        return;
    }
    getchar(); // Limpia el buffer

    if (seleccion > 0 && seleccion <= totalTareas) {
        mostrarDetalleTarea(&tareas[seleccion - 1]); // Llama a mostrarDetalleTarea para la tarea seleccionada
    } else if (seleccion != 0) {
        printf("Selección fuera de rango.\n");
    }
    // --- FIN NUEVA FUNCIONALIDAD ---
}

// Muestra tareas según un filtro de estado específico.
void verTareasFiltradas(const char *filtro) {
    limpiarPantalla();
    printf("--- Tareas %s ---\n\n", filtro);
    int encontradas = 0;
    int indices_encontrados[MAX_TAREAS]; // Guardar índices para posible selección posterior

    for (int i = 0; i < totalTareas; i++) {
        if (strcmp(tareas[i].estado, filtro) == 0) {
            printf("[%d] Título: %s | Estado: %s | Vencimiento: %s\n", 
                   encontradas + 1, tareas[i].titulo, tareas[i].estado, tareas[i].vencimiento);
            indices_encontrados[encontradas] = i; // Guardar el índice real en el array tareas
            encontradas++;
        }
    }
    if (encontradas == 0) {
        printf("No hay tareas %s.\n", filtro);
        return; // Salir si no hay tareas para seleccionar
    }

    // Permitir seleccionar tarea también en filtros específicos
    printf("\n¿Deseas ver los detalles o editar alguna de estas tareas?\nIntroduce el número o 0 para volver.\n> ");
    int seleccion;
    if (scanf("%d", &seleccion) != 1) {
        printf("Entrada no válida. Volviendo al menú anterior.\n");
        while (getchar() != '\n');
        return;
    }
    getchar(); // Limpia el buffer

    if (seleccion > 0 && seleccion <= encontradas) {
        mostrarDetalleTarea(&tareas[indices_encontrados[seleccion - 1]]); 
    } else if (seleccion != 0) {
        printf("Selección fuera de rango.\n");
    }
}


// Permite al usuario crear una nueva tarea.
void crearTarea() {
    if (totalTareas >= MAX_TAREAS) {
        printf("Lo siento, no se pueden agregar más tareas. Límite alcanzado.\n");
        return;
    }

    Tarea *nuevaTarea = &tareas[totalTareas]; // Apunta a la siguiente posición libre en el array
    char input[501]; // Buffer para la entrada del usuario, ajustado al tamaño máximo de descripción

    limpiarPantalla();
    printf("--- Creando Nueva Tarea ---\n\n");

    printf("Título de la tarea (máx. 100 caracteres):\n> "); // Mensaje actualizado
    // fgets es más seguro que scanf para leer strings porque evita desbordamientos de buffer.
    fgets(nuevaTarea->titulo, sizeof(nuevaTarea->titulo), stdin);
    // Elimina el salto de línea que fgets añade al final de la cadena
    nuevaTarea->titulo[strcspn(nuevaTarea->titulo, "\n")] = 0;

    printf("Descripción (máx. 500 caracteres):\n> "); // Mensaje actualizado
    fgets(nuevaTarea->descripcion, sizeof(nuevaTarea->descripcion), stdin);
    nuevaTarea->descripcion[strcspn(nuevaTarea->descripcion, "\n")] = 0;

    // Establece el estado inicial y la fecha de creación automáticamente
    strcpy(nuevaTarea->estado, "Pendiente");
    strcpy(nuevaTarea->dificultad, "+--"); // Dificultad por defecto

    time_t now = time(NULL); // Obtiene el tiempo actual
    struct tm *tm_info = localtime(&now); // Convierte el tiempo a estructura tm
    // Formatea la fecha actual en "DD/MM/AAAA"
    strftime(nuevaTarea->creacion, sizeof(nuevaTarea->creacion), "%d/%m/%Y", tm_info);

    printf("Fecha de vencimiento (DD/MM/AAAA):\n> ");
    fgets(input, sizeof(input), stdin);
    input[strcspn(input, "\n")] = 0;
    strcpy(nuevaTarea->vencimiento, input);

    totalTareas++; // Incrementa el contador de tareas
    printf("\n¡Tarea '%s' creada exitosamente!\n", nuevaTarea->titulo);
}

// Llama a la función crearTarea. Separada para claridad.
void agregarTareaNueva() {
    crearTarea();
}

// Busca tareas por título (o parte del título) y permite al usuario seleccionar una para ver detalles.
void buscarTareaPorTitulo() {
    char titulo[101]; // Ajustado al nuevo tamaño máximo del título
    limpiarPantalla();
    printf("--- Buscar Tarea por Título ---\n\n");
    printf("Introduce el título o parte del título de la tarea a buscar:\n> ");
    fgets(titulo, sizeof(titulo), stdin);
    titulo[strcspn(titulo, "\n")] = 0; // Elimina el salto de línea

    int encontrados[MAX_TAREAS]; // Array para guardar los índices de las tareas encontradas
    int cantidad = 0; // Contador de tareas encontradas

    // Itera sobre todas las tareas para buscar coincidencias
    for (int i = 0; i < totalTareas; i++) {
        // strstr busca una subcadena. strcasestr (GNU C) sería para búsqueda insensible a mayúsculas/minúsculas.
        // Aquí se usa strstr que es sensible.
        if (strstr(tareas[i].titulo, titulo)) { 
            encontrados[cantidad++] = i; // Almacena el índice de la tarea encontrada
        }
    }

    if (cantidad == 0) {
        printf("\nNo se encontraron tareas relacionadas con '%s'.\n", titulo);
        return;
    }

    printf("\n--- Tareas relacionadas ---\n\n");
    for (int i = 0; i < cantidad; i++) {
        printf(" [%d] %s\n", i + 1, tareas[encontrados[i]].titulo);
    }

    printf("\n¿Deseas ver los detalles de alguna? Introduce el número o 0 para volver.\n> ");
    int seleccion;
    if (scanf("%d", &seleccion) != 1) {
        printf("Entrada no válida. Volviendo al menú anterior.\n");
        while (getchar() != '\n');
        return;
    }
    getchar(); // Limpia el buffer

    // Valida la selección del usuario
    if (seleccion > 0 && seleccion <= cantidad) {
        mostrarDetalleTarea(&tareas[encontrados[seleccion - 1]]); // Muestra los detalles de la tarea seleccionada
    } else if (seleccion != 0) {
        printf("Selección fuera de rango.\n");
    }
}

// Muestra todos los detalles de una tarea específica y permite editarla.
void mostrarDetalleTarea(Tarea *t) {
    limpiarPantalla();
    printf("--- Detalle de la Tarea ---\n\n");
    printf("  Título:       %s\n", t->titulo);
    printf("  Descripción:  %s\n", t->descripcion);
    printf("  Estado:       %s\n", t->estado);
    printf("  Dificultad:   %s\n", t->dificultad);
    printf("  Vencimiento:  %s\n", t->vencimiento);
    printf("  Creación:     %s\n", t->creacion);

    printf("\nSi deseas editarla, presiona 'E'. Presiona '0' para volver.\n> ");
    char input;
    // Lee un solo carácter y lo limpia del buffer
    if (scanf(" %c", &input) != 1) { // Nota el espacio antes de %c para consumir el '\n' pendiente
        printf("Entrada no válida. Volviendo al menú anterior.\n");
        while (getchar() != '\n');
        return;
    }
    getchar(); // Limpia el buffer

    if (input == 'E' || input == 'e') {
        editarTarea(t); // Llama a la función de edición si el usuario lo desea
    }
}

// Permite editar los atributos de una tarea.
void editarTarea(Tarea *t) {
    limpiarPantalla();
    printf("--- Editando Tarea: %s ---\n\n", t->titulo);
    printf("- Si deseas mantener los valores de un atributo, simplemente déjalo en blanco (presiona ENTER).\n");
    printf("- Si deseas dejar en blanco un atributo, escribe un espacio y luego ENTER.\n\n");

    char input[501]; // Buffer para la entrada del usuario, ajustado al tamaño máximo de descripción

    // Edición de la descripción
    printf("1. Ingresa la nueva descripción (máx. 500 caracteres, actual: %s):\n> ", t->descripcion); // Mensaje actualizado
    fgets(input, sizeof(input), stdin);
    input[strcspn(input, "\n")] = 0;
    if (strcmp(input, "") != 0) { // Si el input no está vacío, actualiza
        strcpy(t->descripcion, input);
    }

    // Edición del estado
    printf("2. Estado actual: %s\n", t->estado);
    printf("   Nuevo estado ([P]endiente / [E]n curso / [T]erminada / [C]ancelada):\n> ");
    fgets(input, sizeof(input), stdin);
    input[strcspn(input, "\n")] = 0;
    if (strlen(input) == 1) { // Si se ingresó un solo carácter
        switch (input[0]) {
            case 'P': case 'p': strcpy(t->estado, "Pendiente"); break;
            case 'E': case 'e': strcpy(t->estado, "En curso"); break;
            case 'T': case 't': strcpy(t->estado, "Terminada"); break;
            case 'C': case 'c': strcpy(t->estado, "Cancelada"); break;
            default: printf("Opción de estado no válida. Estado no modificado.\n"); break;
        }
    } else if (strlen(input) > 1) {
        printf("Entrada de estado no válida. Estado no modificado.\n");
    }


    // Edición de la dificultad
    printf("3. Dificultad actual: %s\n", t->dificultad);
    printf("   Nueva dificultad ([1] muy fácil / [2] media / [3] muy difícil):\n> ");
    fgets(input, sizeof(input), stdin);
    input[strcspn(input, "\n")] = 0;
    if (strcmp(input, "1") == 0) strcpy(t->dificultad, "+--");
    else if (strcmp(input, "2") == 0) strcpy(t->dificultad, "++-");
    else if (strcmp(input, "3") == 0) strcpy(t->dificultad, "+++");
    else if (strcmp(input, "") != 0) { // Si no es vacío y no es 1, 2 o 3
        printf("Opción de dificultad no válida. Dificultad no modificada.\n");
    }


    // Edición de la fecha de vencimiento
    printf("4. Vencimiento actual: %s\n", t->vencimiento);
    printf("   Nuevo vencimiento (DD/MM/AAAA):\n> ");
    fgets(input, sizeof(input), stdin);
    input[strcspn(input, "\n")] = 0;
    if (strcmp(input, "") != 0) {
        strcpy(t->vencimiento, input);
    }

    printf("\n¡Datos guardados!\n");
    printf("Presiona cualquier tecla para continuar...");
    getchar(); // Espera una entrada del usuario antes de volver
}

// Función para eliminar una tarea (aún no completamente integrada en el flujo)
void eliminarTarea(int indice) {
    if (indice < 0 || indice >= totalTareas) {
        printf("Índice de tarea inválido para eliminar.\n");
        return;
    }

    // Mueve todas las tareas posteriores una posición hacia atrás
    for (int i = indice; i < totalTareas - 1; i++) {
        tareas[i] = tareas[i+1];
    }
    totalTareas--; // Decrementa el contador de tareas

    printf("Tarea eliminada exitosamente.\n");
}