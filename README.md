2) Exports por sesión
2.1 Primario (ATL)
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/app/oracle/product/19.3.0/dbhome_1
export ORACLE_SID=C1ATLANTA1      # en n2: C1ATLANTA2
export PATH=$ORACLE_HOME/bin:$PATH
export TNS_ADMIN=$ORACLE_HOME/network/admin

2.2 Standby (PAR)
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/app/oracle/product/19.3.0/dbhome_1
export ORACLE_SID=C1PARIS1
export PATH=$ORACLE_HOME/bin:$PATH
export TNS_ADMIN=$ORACLE_HOME/network/admin

2.3 Usuario Grid (GI)
su - grid
export GRID_HOME=/u01/app/19.3.0/grid
export PATH=$GRID_HOME/bin:$PATH

3) tnsnames.ora en todos los nodos

Ruta: $ORACLE_HOME/network/admin/tnsnames.ora

C1ATLANTA =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = c1-atl-scan.example.com)(PORT = 1521))
    (CONNECT_DATA = (SERVICE_NAME = C1ATLANTA))
  )

C1PARIS =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = c1-par-scan.example.com)(PORT = 1521))
    (CONNECT_DATA = (SERVICE_NAME = C1PARIS))
  )

-- Conectar al standby en NOMOUNT/MOUNT por SID (auxiliar RMAN)
C1PARIS_AUX =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = c1-par-n1.example.com)(PORT = 1521))
    (CONNECT_DATA = (SID = C1PARIS1))
  )


Pruebas:

tnsping C1ATLANTA
tnsping C1PARIS
tnsping C1PARIS_AUX


Usa el alias TNS C1ATLANTA como target en RMAN (no easy-connect).

4) Listener GI (PAR) – Entrada estática + reinicio con srvctl

Editar (grid):

su - grid
vi /u01/app/19.3.0/grid/network/admin/listener.ora


Agregar bloque:

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = C1PARIS_AUX)
      (ORACLE_HOME  = /u01/app/oracle/product/19.3.0/dbhome_1)
      (SID_NAME     = C1PARIS1)
    )
  )


Reiniciar con GI:
