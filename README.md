# Convenciones de Codificación Aplicadas
## 1. Nombres: 
Los nombres de las variables y funciones son descriptivos y siguen las convenciones de estilo camelCase para métodos y snake_case para variables.

```python
class Voto(db.Model):
    __tablename__ = 'voto'
    id_voto = db.Column(db.Integer, primary_key=True, autoincrement=True)
    id_elector = db.Column(db.Integer, db.ForeignKey('elector.id'), nullable=True)
    id_lista = db.Column(db.Integer, db.ForeignKey('lista_candidato.id_lista'), nullable=True)

    def __init__(self, id_elector, id_lista):
        self.id_elector = id_elector
        self.id_lista = id_lista
```
## 2. Funciones
Las funciones son cortas y realizan una única tarea.
```python
@home_bp.route('/ListasCandidatos', methods=['GET'])
def listar_candidatos():
    elecciones_json = eleccion_servicio.get_all_eleccion()
    return render_template('ListaCandidato/lista_candidatos.html', elecciones=elecciones_json)
```
## 3. Comentarios
Se usan comentarios para explicar secciones complejas del código.
```python
from abc import ABC, abstractmethod

class IEleccionServicio(ABC):
    @abstractmethod
    def get_all_eleccion(self):
        """Método abstracto para obtener todas las elecciones"""
        pass
```
## 4. Estructura de Código
El código está organizado en módulos y clases según su funcionalidad, siguiendo el principio de responsabilidad única.
```python
class Candidato(db.Model):
    __tablename__ = 'candidato'
    id = db.Column(db.Integer, primary_key=True)
    nombres = db.Column(db.String(100), nullable=False)
    apellido_paterno = db.Column(db.String(100), nullable=False)
    apellido_materno = db.Column(db.String(100), nullable=False)
    id_lista_candidato = db.Column(db.Integer, db.ForeignKey('lista_candidato.id_lista'), nullable=True)

    def __init__(self, nombres, apellido_paterno, apellido_materno, id_lista_canditado):
        self.nombres = nombres
        self.apellido_paterno = apellido_paterno
        self.apellido_materno = apellido_materno
        self.id_lista_candidato = id_lista_canditado
```
`Corrección de Problemas con SonarLint`

# Principios SOLID Aplicados

## Principio de Responsabilidad Única (Single Responsibility Principle - SRP)
Cada clase en este proyecto tiene una única responsabilidad. Por ejemplo, la clase `Voto` se encarga únicamente de representar la entidad del voto.

```python
from app import db

class Voto(db.Model):
    __tablename__ = 'voto'
    id_voto = db.Column(db.Integer, primary_key=True, autoincrement=True)
    id_elector = db.Column(db.Integer, db.ForeignKey('elector.id'), nullable=True)
    id_lista = db.Column(db.Integer, db.ForeignKey('lista_candidato.id_lista'), nullable=True)

    def __init__(self, id_elector, id_lista):
        self.id_elector = id_elector
        self.id_lista = id_lista
```
## Principio de Abierto/Cerrado (Open/Closed Principle - OCP)
Las clases están abiertas para extensión pero cerradas para modificación. Por ejemplo, Candidato puede ser extendida sin modificar su implementación existente.
from app import db

```python
class Candidato(db.Model):
    __tablename__ = 'candidato'
    id = db.Column(db.Integer, primary_key=True)
    nombres = db.Column(db.String(100), nullable=False)
    apellido_paterno = db.Column(db.String(100), nullable=False)
    apellido_materno = db.Column(db.String(100), nullable=False)
    id_lista_candidato = db.Column(db.Integer, db.ForeignKey('lista_candidato.id_lista'), nullable=True)
    
    def __init__(self, nombres, apellido_paterno, apellido_materno, id_lista_canditado):
        self.nombres = nombres
        self.apellido_paterno = apellido_paterno
        self.apellido_materno = apellido_materno
        self.id_lista_candidato = id_lista_canditado
```

## Principio de Inversión de Dependencia (Dependency Inversion Principle - DIP)
Utilizamos abstracciones en lugar de concretar implementaciones para permitir una fácil modificación y prueba. Por ejemplo, el servicio IEleccionServicio se define como una interfaz abstracta.

```python
from abc import ABC, abstractmethod

class IEleccionServicio(ABC):
    @abstractmethod
    def get_all_eleccion(self):
        pass

@home_bp.route('/ListasCandidatos', methods=['GET'])
def listar_candidatos():
    elecciones_json = eleccion_servicio.get_all_eleccion()
    return render_template('ListaCandidato/lista_candidatos.html', elecciones=elecciones_json)
```

# Estilos de Programación Utilizados
## 1. Estilo Cookbook
Este estilo implica el uso de recetas de código reutilizables para tareas comunes. Un ejemplo es la implementación de modelos y esquemas utilizando SQLAlchemy y Marshmallow.

Fragmento de Código:

```python
Copiar código
from app import db
from app import ma

class Voto(db.Model):
    __tablename__ = 'voto'
    id_voto = db.Column(db.Integer, primary_key=True, autoincrement=True)
    id_elector = db.Column(db.Integer, db.ForeignKey('elector.id'), nullable=True)
    id_lista = db.Column(db.Integer, db.ForeignKey('listacandidato.id_lista'), nullable=True)

    def __init__(self, id_elector, id_lista):
        self.id_elector = id_elector
        self.id_lista = id_lista

class VotoSchema(ma.Schema):
    class Meta:
        fields = ('id_voto', 'id_elector', 'id_lista_candidato')
```

## 2. Estilo Error/Exception Handling
Este estilo se enfoca en la gestión adecuada de errores y excepciones para asegurar que el sistema sea robusto y manejable.

Fragmento de Código:

```python
Copiar código
class VotoServicioImpl(IVotoServicio):
        
    def get_voto_by_elector(self, id_elector):
        try:
            voto = db.session.query(Elector.nombres).join(Voto, Elector.id == Voto.id_elector).filter(Elector.id == id_elector).all()
            result = [{"nombre": tupla[0]} for tupla in voto]
            return result
        except Exception as e:
            logger.error(f'Error al obtener el voto del elector: {str(e)}')
            raise e
    def votar(self, id_lista, id_elector):
        try:
            voto = Voto(id_elector, id_lista)
            db.session.add(voto)
            db.session.commit()
            logger.info(f'Voto registrado correctamente: {voto}')
        except Exception as e:
            db.session.rollback()
            logger.error(f'Error al registrar el voto: {str(e)}')
            raise e
    def get_all_votos(self):
        votos = db.session.query(
            Elector.nombres,
            Elector.apellido_paterno,
            Elector.apellido_materno,
            ListaCandidato.nombre
        ).join(
            Voto, Elector.id == Voto.id_elector
        ).join(
            ListaCandidato, ListaCandidato.id_lista == Voto.id_lista
        ).all()
        
        result = [
            {
                "nombre_completo": f"{tupla[0]} {tupla[1]} {tupla[2]}",
                "nombre_lista": tupla[3]
            }
            for tupla in votos
        ]
        return result
```

## 3. Estilo Restful
Este estilo implica la creación de servicios RESTful para permitir la interacción con el sistema a través de HTTP, siguiendo los principios de REST.

Fragmento de Código:

```python
Copiar código
@app.route('/elecciones', methods=['GET'])
def get_elecciones():
    try:
        elecciones = EleccionServicioImpl().get_all_eleccion()
        return jsonify(elecciones), 200
    except Exception as e:
        return jsonify({'error': str(e)}), 500
```

Copiar código
## Estilos de Programación Utilizados

### Estilo Cookbook
**Descripción:** Uso de recetas de código reutilizables.
**Fragmento de Código:**
```python
from app import db
from app import ma

class Voto(db.Model):
    __tablename__ = 'voto'
    id_voto = db.Column(db.Integer, primary_key=True, autoincrement=True)
    id_elector = db.Column(db.Integer, db.ForeignKey('elector.id'), nullable=True)
    id_lista = db.Column(db.Integer, db.ForeignKey('listacandidato.id_lista'), nullable=True)

    def __init__(self, id_elector, id_lista):
        self.id_elector = id_elector
        self.id_lista = id_lista

class VotoSchema(ma.Schema):
    class Meta:
        fields = ('id_voto', 'id_elector', 'id_lista_candidato')
```

## Estilo Error/Exception Handling
Descripción: Gestión adecuada de errores y excepciones.
Fragmento de Código:

```python
Copiar código
class EleccionServicioImpl(IEleccionServicio):
    def get_all_eleccion(self):
        try:
            all_eleccion = Eleccion.query.all()
            result = eleccion_schemas.dump(all_eleccion)
            return result
        except Exception as e:
            logger.error(f'Error al obtener todas las elecciones: {str(e)}')
            raise e
```

## Estilo Restful
Descripción: Creación de servicios RESTful.
Fragmento de Código:

```python
Copiar código
@app.route('/elecciones', methods=['GET'])
def get_elecciones():
    try:
        elecciones = EleccionServicioImpl().get_all_eleccion()
        return jsonify(elecciones), 200
    except Exception as e:
        return jsonify({'error': str(e)}), 500
```