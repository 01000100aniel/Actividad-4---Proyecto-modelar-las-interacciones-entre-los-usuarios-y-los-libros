"""
Estructuras de datos (JAVIER LUNA 25082025_C12_202534)
Unidad de Aprendizaje II
biblioteca_arboles.(Daniel Steven Alvarez Ch)
Sistema de gestión de biblioteca actualizado:
- Árbol Binario de Búsqueda (ABB) para libros y usuarios (búsqueda eficiente).
- Árbol General para representar la jerarquía de la biblioteca (visualización).
- Cola para préstamo de libros.
"""

# -------------------------
# Modelos de dominio
# -------------------------
class Libro:
    def __init__(self, id_libro, titulo, autor, año, genero="General", estado="disponible"):
        self.id = id_libro
        self.titulo = titulo
        self.autor = autor
        self.año = año
        self.genero = genero
        self.estado = estado

    def __str__(self):
        return f"[{self.id}] {self.titulo} - {self.autor} ({self.año}) - {self.genero} - {self.estado}"


class Usuario:
    def __init__(self, id_usuario, nombre, contacto):
        self.id = id_usuario
        self.nombre = nombre
        self.contacto = contacto
        self.libros_prestados = []

    def __str__(self):
        return f"[{self.id}] {self.nombre} - {self.contacto} - Libros prestados: {len(self.libros_prestados)}"


class Prestamo:
    def __init__(self, id_prestamo, usuario, libro):
        self.id = id_prestamo
        self.usuario = usuario
        self.libro = libro

    def __str__(self):
        return f"Préstamo ID: {self.id} - Usuario: {self.usuario.nombre} - Libro: {self.libro.titulo}"


# -------------------------
# Árbol Binario de Búsqueda (ABB) - para búsquedas eficientes
# -------------------------
def _llave_de_id(dato):
    """
    Intenta convertir id a int para comparaciones numéricas,
    si falla usa la cadena tal cual.
    """
    try:
        return int(dato.id)
    except Exception:
        return dato.id


class NodoABB:
    def __init__(self, dato):
        self.dato = dato
        self.izq = None
        self.der = None


class ArbolBinarioBusqueda:
    def __init__(self):
        self.raiz = None

    def insertar(self, dato):
        if self.raiz is None:
            self.raiz = NodoABB(dato)
            return True
        else:
            return self._insertar_rec(self.raiz, dato)

    def _insertar_rec(self, nodo, dato):
        key_nuevo = _llave_de_id(dato)
        key_nodo = _llave_de_id(nodo.dato)
        if key_nuevo == key_nodo:
            # ya existe el id
            return False
        elif key_nuevo < key_nodo:
            if nodo.izq:
                return self._insertar_rec(nodo.izq, dato)
            else:
                nodo.izq = NodoABB(dato)
                return True
        else:
            if nodo.der:
                return self._insertar_rec(nodo.der, dato)
            else:
                nodo.der = NodoABB(dato)
                return True

    def buscar(self, id_buscar):
        try:
            key = int(id_buscar)
        except:
            key = id_buscar
        return self._buscar_rec(self.raiz, key)

    def _buscar_rec(self, nodo, key):
        if nodo is None:
            return None
        key_nodo = _llave_de_id(nodo.dato)
        if key == key_nodo:
            return nodo.dato
        elif key < key_nodo:
            return self._buscar_rec(nodo.izq, key)
        else:
            return self._buscar_rec(nodo.der, key)

    def _min_nodo(self, nodo):
        actual = nodo
        while actual.izq:
            actual = actual.izq
        return actual

    def eliminar(self, id_eliminar):
        try:
            key = int(id_eliminar)
        except:
            key = id_eliminar
        self.raiz, eliminado = self._eliminar_rec(self.raiz, key)
        return eliminado

    def _eliminar_rec(self, nodo, key):
        if nodo is None:
            return nodo, False
        key_nodo = _llave_de_id(nodo.dato)
        if key < key_nodo:
            nodo.izq, eliminado = self._eliminar_rec(nodo.izq, key)
            return nodo, eliminado
        elif key > key_nodo:
            nodo.der, eliminado = self._eliminar_rec(nodo.der, key)
            return nodo, eliminado
        else:
            # encontrado
            if nodo.izq is None:
                return nodo.der, True
            elif nodo.der is None:
                return nodo.izq, True
            else:
                sucesor = self._min_nodo(nodo.der)
                nodo.dato = sucesor.dato
                nodo.der, _ = self._eliminar_rec(nodo.der, _llave_de_id(sucesor.dato))
                return nodo, True

    def mostrar_inorden(self, nodo=None):
        if self.raiz is None:
            print("(vacío)")
            return
        if nodo is None:
            nodo = self.raiz
        if nodo.izq:
            self.mostrar_inorden(nodo.izq)
        print(nodo.dato)
        if nodo.der:
            self.mostrar_inorden(nodo.der)


# -------------------------
# Árbol General para jerarquía (ej. Biblioteca -> Libros -> Géneros -> ...)
# -------------------------
class NodoGeneral:
    def __init__(self, nombre, referencia=None):
        """
        nombre: texto que describe el nodo (ej. "Libros", "Usuarios", "Ficción")
        referencia: opcional, puede apuntar a un objeto real (Libro o Usuario)
        """
        self.nombre = nombre
        self.referencia = referencia
        self.hijos = []

    def agregar_hijo(self, nodo_hijo):
        self.hijos.append(nodo_hijo)

    def encontrar(self, texto):
        """Busca un nodo cuyo nombre coincida exactamente (primera coincidencia, DFS)."""
        if self.nombre == texto:
            return self
        for h in self.hijos:
            r = h.encontrar(texto)
            if r:
                return r
        return None

    def mostrar(self, nivel=0):
        pref = "  " * nivel
        if self.referencia:
            print(f"{pref}- {self.nombre} -> ({self.referencia})")
        else:
            print(f"{pref}- {self.nombre}")
        for hijo in self.hijos:
            hijo.mostrar(nivel + 1)


# -------------------------
# Cola simple para préstamos
# -------------------------
class Cola:
    def __init__(self):
        self.items = []

    def encolar(self, item):
        self.items.append(item)

    def desencolar(self):
        if self.items:
            return self.items.pop(0)
        return None

    def esta_vacia(self):
        return len(self.items) == 0

    def mostrar(self):
        if self.esta_vacia():
            print("No hay préstamos en cola.")
        else:
            for prestamo in self.items:
                print(prestamo)


# -------------------------
# Sistema de Biblioteca (integración)
# -------------------------
class SistemaBiblioteca:
    def __init__(self):
        # Estructuras para eficiencia
        self.libros = ArbolBinarioBusqueda()
        self.usuarios = ArbolBinarioBusqueda()
        # Estructura jerárquica para visualización (árbol general)
        self.arbol_jerarquico = NodoGeneral("Biblioteca")
        nodo_libros = NodoGeneral("Libros")
        nodo_usuarios = NodoGeneral("Usuarios")
        self.arbol_jerarquico.agregar_hijo(nodo_libros)
        self.arbol_jerarquico.agregar_hijo(nodo_usuarios)
        # Cola de prestamos
        self.prestamos = Cola()
        self.contador_prestamos = 1

    # -----------------
    # Helper: buscar ubicación en árbol general por género (o crear)
    # -----------------
    def _ubicacion_libro_en_jerarquia(self, libro: Libro):
        nodo_libros = self.arbol_jerarquico.encontrar("Libros")
        # intentamos agrupar por género
        genero = libro.genero if libro.genero else "General"
        nodo_genero = nodo_libros.encontrar(genero)
        if nodo_genero is None:
            nodo_genero = NodoGeneral(genero)
            nodo_libros.agregar_hijo(nodo_genero)
        # finalmente agregamos el libro como nodo hoja con referencia al objeto
        nodo_libro = NodoGeneral(f"{libro.titulo} [{libro.id}]", referencia=libro)
        nodo_genero.agregar_hijo(nodo_libro)

    def _ubicacion_usuario_en_jerarquia(self, usuario: Usuario):
        nodo_usuarios = self.arbol_jerarquico.encontrar("Usuarios")
        # podemos agrupar por inicial del nombre
        inicial = usuario.nombre.strip()[0].upper() if usuario.nombre else "SinNombre"
        nodo_inicial = nodo_usuarios.encontrar(inicial)
        if nodo_inicial is None:
            nodo_inicial = NodoGeneral(inicial)
            nodo_usuarios.agregar_hijo(nodo_inicial)
        nodo_usuario = NodoGeneral(f"{usuario.nombre} [{usuario.id}]", referencia=usuario)
        nodo_inicial.agregar_hijo(nodo_usuario)

    # -----------------
    # Registro
    # -----------------
    def registrar_libro_obj(self, libro: Libro):
        # inserta en ABB y en árbol general
        ok = self.libros.insertar(libro)
        if not ok:
            return False, "ID de libro ya existe."
        self._ubicacion_libro_en_jerarquia(libro)
        return True, "Libro registrado con éxito."

    def registrar_usuario_obj(self, usuario: Usuario):
        ok = self.usuarios.insertar(usuario)
        if not ok:
            return False, "ID de usuario ya existe."
        self._ubicacion_usuario_en_jerarquia(usuario)
        return True, "Usuario registrado con éxito."

    # Métodos interactivos para CLI
    def registrar_libro(self):
        id_libro = input("ID libro: ").strip()
        if self.libros.buscar(id_libro):
            print("Este libro ya está registrado.")
            return
        titulo = input("Título: ").strip()
        autor = input("Autor: ").strip()
        año = input("Año: ").strip()
        genero = input("Género (opcional): ").strip()
        libro = Libro(id_libro, titulo, autor, año, genero if genero else "General")
        ok, msg = self.registrar_libro_obj(libro)
        print(msg)

    def registrar_usuario(self):
        id_usuario = input("ID usuario: ").strip()
        if self.usuarios.buscar(id_usuario):
            print("Este usuario ya está registrado.")
            return
        nombre = input("Nombre: ").strip()
        contacto = input("Contacto: ").strip()
        usuario = Usuario(id_usuario, nombre, contacto)
        ok, msg = self.registrar_usuario_obj(usuario)
        print(msg)

    # -----------------
    # Préstamo
    # -----------------
    def prestar_libro(self):
        id_usuario = input("ID usuario: ").strip()
        usuario = self.usuarios.buscar(id_usuario)
        if not usuario:
            print("Usuario no encontrado.")
            return
        id_libro = input("ID libro: ").strip()
        libro = self.libros.buscar(id_libro)
        if not libro:
            print("Libro no encontrado.")
            return
        if libro.estado != "disponible":
            print("Libro no está disponible para préstamo.")
            return
        # Cambiar estado libro y agregar préstamo
        libro.estado = "prestado"
        usuario.libros_prestados.append(libro)
        prestamo = Prestamo(self.contador_prestamos, usuario, libro)
        self.prestamos.encolar(prestamo)
        self.contador_prestamos += 1
        print(f"Préstamo realizado con éxito: {prestamo}")

    # -----------------
    # Devolución
    # -----------------
    def devolver_libro(self):
        id_usuario = input("ID usuario: ").strip()
        usuario = self.usuarios.buscar(id_usuario)
        if not usuario:
            print("Usuario no encontrado.")
            return
        id_libro = input("ID libro: ").strip()
        libro = self.libros.buscar(id_libro)
        if not libro:
            print("Libro no encontrado.")
            return
        if libro not in usuario.libros_prestados:
            print("Este libro no está prestado a este usuario.")
            return
        libro.estado = "disponible"
        usuario.libros_prestados.remove(libro)
        print(f"Libro '{libro.titulo}' devuelto con éxito.")

    # -----------------
    # Mostrar info
    # -----------------
    def mostrar_libros_ordenados(self):
        print("\nLibros (ordenados por ID):")
        self.libros.mostrar_inorden()

    def mostrar_usuarios_ordenados(self):
        print("\nUsuarios (ordenados por ID):")
        self.usuarios.mostrar_inorden()

    def mostrar_prestamos(self):
        print("\nPréstamos pendientes:")
        self.prestamos.mostrar()

    def mostrar_jerarquia(self):
        print("\nEstructura jerárquica de la Biblioteca:")
        self.arbol_jerarquico.mostrar()


# -------------------------
# Menú de interacción
# -------------------------
def menu():
    sistema = SistemaBiblioteca()

    # ejemplo rápido de datos iniciales (puedes comentar si no los quieres)
    ejemplo_libros = [
        Libro("100", "Cien Años de Soledad", "Gabriel García Márquez", "1967", genero="Ficción"),
        Libro("200", "Introducción a Algoritmos", "Cormen et al.", "2009", genero="Ciencia"),
        Libro("150", "La Odisea", "Homero", "800 A.C.", genero="Clásicos"),
    ]
    ejemplo_usuarios = [
        Usuario("1", "Ana Pérez", "ana@example.com"),
        Usuario("2", "Juan Gómez", "juan@example.com"),
    ]
    for lb in ejemplo_libros:
        sistema.registrar_libro_obj(lb)
    for us in ejemplo_usuarios:
        sistema.registrar_usuario_obj(us)

    while True:
        print("\n--- Sistema de Gestión de Biblioteca (Con Árboles) ---")
        print("1. Registrar libro")
        print("2. Registrar usuario")
        print("3. Prestar libro")
        print("4. Devolver libro")
        print("5. Mostrar libros (ordenados por ID)")
        print("6. Mostrar usuarios (ordenados por ID)")
        print("7. Mostrar préstamos")
        print("8. Mostrar jerarquía (árbol general)")
        print("0. Salir")
        opcion = input("Seleccione una opción: ").strip()

        if opcion == "1":
            sistema.registrar_libro()
        elif opcion == "2":
            sistema.registrar_usuario()
        elif opcion == "3":
            sistema.prestar_libro()
        elif opcion == "4":
            sistema.devolver_libro()
        elif opcion == "5":
            sistema.mostrar_libros_ordenados()
        elif opcion == "6":
            sistema.mostrar_usuarios_ordenados()
        elif opcion == "7":
            sistema.mostrar_prestamos()
        elif opcion == "8":
            sistema.mostrar_jerarquia()
        elif opcion == "0":
            print("¡Hasta luego! Gracias por usar el sistema.")
            break
        else:
            print("Opción no válida, intenta de nuevo.")


if __name__ == "__main__":
    menu()

