# Estructura de Systema Fat32

Sector = default 512B
Cluster = sector x SectorPerCluster, defauls 8KB

struct MBR {
	BytsPerSec = 512 Bytes
	SecPerClus = 16 Sectors
	RsvdSecCnt = (Boot Sec + all reserved Sec)
	NumFats = 2 Tablas de Fat32
	FatSz32 = (Size of Tabla Fat32) in Sectors
	RootCluster = el cluster donde esta la raiz del disco "/"
}

struct Dir {
	(Dir_Item | Dir_longName) *items;
}

struct Dir_Item {
	char name[8];			// empty fill with space
	char extension[3]		// empty & directory fill with space
	byte attribute[1]		// 
	byte reserved-NT[1]		// Reservado para sistemas NT, se suele dejar en 0
	byte Create_MS[1] 		// el milisegundo en el que se creo
	byte Create_time[2]		// la hora en la que se creo
	byte Create_date[2]		// la fecha en la que se creo
	byte Access_date[2]		// la fecha del ultimo acceso
	byte Cluster_High[2]	// Los 2 primeros Bytes del cluster que pertenece este archivo
	byte Modify_time[2]		// la hora en la que se modifico
	byte Modify_date[2]		// la fecha en la que se modifico
	byte Cluster_LOW[2]		// Los 2 ultimos Bytes del cluster que pertenece este archivo
	byte FileSize[4]		// tamaño del archivo, para las direcciones se suele figar en 0
}

// se instertan del reves desde el ultimo hasta el primerpo y luego se añade el short name
struct Dir_longName {
	byte ULFN[1]			// flags de posicion y mas cosas
	char name[10];			// 
	byte attribute[1]		// 0x0F => Long File Name
	byte reserved[1]		// 0x00
	byte CheckSum[1] 		// Check Sum
	char name[12]			// 
	byte reserved[2]		// 0x00
	char name[4]			// 
}

/*
	Dir_Item.attribute:
		- 0x00: Array Empty
		- 0x08: VolumewId
		- 0x10: Directory
		- 0x20: Archive
		- 0x0F: Long File Name

*/


Calculation: {
	DataFirstSec = RsvdSecCnt + (NumFats * FatSz32)
	ClusterSec = DataFirstSec + ((Cluster_ID - 2) * SecPerClus)	// -2 es porque los 2 primeros clusters no se usan
	ClusterAddr = ClusterSec * BytsPerSec
}

## Tabla FAT32
- es un array que tiene un item para cada cluster
	- Item[0]: no indica el cluster, si no propiedades de la memoria
	- Item[1]: no indica el cluster, si no propiedades del sistema operativo
	- Item Value:
		- 0x00000000: cluster are empty
		- 0x0FFFFFF7: damaged Cluster
		- 0x0FFFFFF8: -> 0x0FFFFFFF, End Of File
		- other number: blockchain (next cluster)

