# tarea# Sistema de Información para Cajero Automático (ATM)

## CE1. Implementación de Sistemas de Información (OLTP & BI)

### 1. Modelo Relacional OLTP (SQL Server / SSMS)

La base de datos `ATM_DB` garantiza la integridad de los datos, el control de concurrencia y la seguridad transaccional mediante restricciones de clave, hash de PIN y procedimientos almacenados con control de excepciones ACID.

#### Script de Creación de Base de Datos y Tablas
```sql
CREATE DATABASE ATM_DB;
GO

USE ATM_DB;
GO

-- Tabla: Cliente
CREATE TABLE Cliente (
    ClienteID INT IDENTITY(1,1) PRIMARY KEY,
    Nombre VARCHAR(50) NOT NULL,
    Apellido VARCHAR(50) NOT NULL,
    DNI CHAR(8) NOT NULL UNIQUE,
    Telefono VARCHAR(15),
    Email VARCHAR(100),
    FechaRegistro DATETIME DEFAULT GETDATE()
);

-- Tabla: Cuenta
CREATE TABLE Cuenta (
    CuentaID INT IDENTITY(1,1) PRIMARY KEY,
    ClienteID INT NOT NULL,
    NumeroCuenta VARCHAR(20) NOT NULL UNIQUE,
    TipoCuenta VARCHAR(20) CHECK (TipoCuenta IN ('Ahorros', 'Corriente')),
    Saldo DECIMAL(12,2) NOT NULL DEFAULT 0.00,
    Estado VARCHAR(15) CHECK (Estado IN ('Activa', 'Bloqueada', 'Inactiva')) DEFAULT 'Activa',
    CONSTRAINT FK_Cuenta_Cliente FOREIGN KEY (ClienteID) REFERENCES Cliente(ClienteID)
);

-- Tabla: Tarjeta
CREATE TABLE Tarjeta (
    TarjetaID INT IDENTITY(1,1) PRIMARY KEY,
    CuentaID INT NOT NULL,
    NumeroTarjeta CHAR(16) NOT NULL UNIQUE,
    PINHash CHAR(64) NOT NULL,
    FechaExpiracion DATE NOT NULL,
    Estado VARCHAR(15) CHECK (Estado IN ('Activa', 'Bloqueada', 'Cancelada')) DEFAULT 'Activa',
    CONSTRAINT FK_Tarjeta_Cuenta FOREIGN KEY (CuentaID) REFERENCES Cuenta(CuentaID)
);

-- Tabla: ATM
CREATE TABLE ATM (
    ATMID INT IDENTITY(1,1) PRIMARY KEY,
    Ubicacion VARCHAR(100) NOT NULL,
    Ciudad VARCHAR(50) NOT NULL,
    EfectivoDisponible DECIMAL(12,2) NOT NULL,
    Estado VARCHAR(15) CHECK (Estado IN ('Operativo', 'Mantenimiento', 'SinEfectivo')) DEFAULT 'Operativo'
);

-- Tabla: TipoTransaccion
CREATE TABLE TipoTransaccion (
    TipoTransaccionID INT IDENTITY(1,1) PRIMARY KEY,
    Nombre VARCHAR(30) NOT NULL UNIQUE
);

-- Tabla: Transaccion
CREATE TABLE Transaccion (
    TransaccionID BIGINT IDENTITY(1,1) PRIMARY KEY,
    TarjetaID INT NOT NULL,
    ATMID INT NOT NULL,
    TipoTransaccionID INT NOT NULL,
    FechaHora DATETIME DEFAULT GETDATE(),
    Monto DECIMAL(10,2) NOT NULL DEFAULT 0.00,
    EstadoTransaccion VARCHAR(15) CHECK (EstadoTransaccion IN ('Exitosa', 'Fallida', 'Cancelada')),
    MontoComision DECIMAL(6,2) DEFAULT 0.00,
    CONSTRAINT FK_Transaccion_Tarjeta FOREIGN KEY (TarjetaID) REFERENCES Tarjeta(TarjetaID),
    CONSTRAINT FK_Transaccion_ATM FOREIGN KEY (ATMID) REFERENCES ATM(ATMID),
    CONSTRAINT FK_Transaccion_Tipo FOREIGN KEY (TipoTransaccionID) REFERENCES TipoTransaccion(TipoTransaccionID)
);
GO
