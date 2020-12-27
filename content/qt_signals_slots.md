Title: Signals y Slots en Qt C++
Date: 2020-12-27 17:43
Category: Programación
Tags: c++,qt
Slug: signals-slots-qt-cpp
Authors: José L. Patiño-Andrés
Summary: Pequeña referencia sobre Signals y Slots en Qt/C++

## Intro

El framework para GUIs Qt dispone de un mecanismo de intercomunicación entre
objetos llamado [_Signals & Slots_](https://doc.qt.io/qt-5/signalsandslots.html).

Esto es especialmente útil para notificar cambios en widgets a otros widgets o
procesos.

Una Signal (señal) es emitida cuando ocurre un evento en particular. Los widgets
(subclases de `QObject`) tienen muchas señales predefinidas para notificar sobre
sus propios eventos y nosotros podemos crear nuevas señales también en esos
widgets o en nuestro propio código.

Un Slot (ranura) es una función que es llamada en respuesta a la emisión de una
señal determinada.

## Uso

Para conectar Signals y Slots, usamos la función `connect` de Qt:

    :::cpp
    connect(Objeto1, signal, Objeto2, slot);

En versiones anteriores a Qt5, había que especificar `SIGNAL` y `SLOT`:

    :::cpp
    connect(Objeto1, SIGNAL(signal), Objeto2, SLOT(slot)

## Ejemplos

### Acción tras la Signal predeterminada de un `QObject`

Supongamos que queremos realizar alguna acción cuando el usuario inserte texto
en un [`QLineEdit`](https://doc.qt.io/qt-5/qlineedit.html). 

Este evento de un usuario escribiendo en un widget `QLineEdit` emite siempre una 
Signal [`textChanged`](https://doc.qt.io/qt-5/qlineedit.html#textChanged). 
Sabiendo esto, podemos añadir un método a nuestra clase para que actúe en base a
la emisión de dicha Signal. Suponiendo que el `QLineEdit` que queremos conectar
es `myLineEdit`:

    :::cpp
    class MyObject : public QObject
    {
        Q_OBJECT

    public:
        explicit MyObject(QWidget *parent = nullptr);
        ~MyObject();

    private slots:
        // Define los slots para este objeto en particular.
        void on_myLineEditChanged(const QString &text);

    private:
       // Más miembros, atributos, etc...
    };

...y ahora conectaremos esa Signal (ese evento) del `QLineEdit` en nuestro
Slot:

    :::cpp
    MyObject::MyObject(QWidget *parent) : QObject(parent)
    {
        // Lo que sea que haga el constructor...
        
        // Conectamos nuestro Slot con la Signal que 
        // emitirá `myLineEdit`.
        connect(myLineEdit, &QLineEdit::textChanged, 
                this, &MyObject::on_myLineEditChanged);
    }

#### Delegación en `QMetaObject::classInfoOffset`

La clase [`QMetaObject`](https://doc.qt.io/qt-5/qmetaobject.html) contiene
metainformación sobre objetos Qt. En concreto, uno de sus miembros públicos es
[`connectSlotsByName`](https://doc.qt.io/qt-5/qmetaobject.html#connectSlotsByName),
que busca Signals recursivamente en todos los objetos descendientes del actual,
conectándolas con Slots que sigan la siguiente nomenclatura:

    :::cpp
    void on_<object name>_<signal name>(<signal parameters>);

Por ejemplo, si hubiéramos nombrado a nuestro Slot:

    :::cpp
    private slots:
        void on_myLineEdit_textChanged();

**No hubiera hecho falta escribir la función `connect`** en nuestro constructor.

### Escribir nuestra propia Signal para notificar un evento

Del mismo modo que hemos definido nuestro Slot para gestionar la emisión de
una Signal por un tercer objeto, podemos crear nuestra propia Signal para que
notifique acerca de un evento en nuestro código a otros objetos que la esperen.

Para ello hemos de definir la Signal en la clase de nuestro objeto:

    :::cpp
    class MyObject : public QObject
    {
        Q_OBJECT

    public:
        explicit MyObject(QWidget *parent = nullptr);
        ~MyObject();

    signals:
        void myObjectCreated(const QString &text);

    private:
        // Más miembros, atributos, etc...
    }

Ahora podemos usar esta Signal de este modo:

    :::cpp
    MyObject::MyObject(QWidget *parent) : QObject(parent)
    {
        // Constructor...

        // Usamos nuestra Signal para notificar el evento.
        emit myObjectCreated(QString("Algo de texto aquí...");
    }
