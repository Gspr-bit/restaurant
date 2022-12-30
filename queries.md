# Consultas

## 1. Platillos y bebidas

1. Listar todos los platillos de una sola categoría en todas las sucursales, (Sólo el administrador general puede usarlo).

   ````json
   db.menu_items.find({
       $and: [
           {"category": "CATEGORY"},
           {
               // If the document has no season
               // or has season and it's currently active
               $or: [
                   {
                       "season_start": {$exists: false}
                   },
                   {
                       $and: [
                           {"season_start": {$lte: new Date()}},
                           {"season_end": {$gte: new Date()}}
                       ]
                   }
               ]
           }
       ]
   })
   ````

   ````json
   db.menu_items.find({$and: [{"category": "CATEGORY"},{$or: [{"season_start": {$exists: false}},{$and: [{"season_start": {$lte: new Date()}},{"season_end": {$gte: new Date()}}]}]}]})
   ````

   

2. Buscar un platillo por nombre en todas las sucursales, (Sólo el administrador general puede usarlo).

   ````json
   db.menu_items.find({
       "name": {
           $regex: 'CADENA*',
           $options: 'i'
       }
   })
   ````

   ````json
   db.menu_items.find({"name": {$regex: 'CADENA*',$options: 'i'}})
   ````

   

3. Listar todos los platillos de una sola categoría en una sucursal.

   ````json
   [
     {
       /**
        * Seleccionar la sucursal
        */
       $match: {
         name: "Morelia",
       },
     },
     {
       /**
        * Seleccionar los documentos de la colección menu_items
        * cuyo id se encuentra en el arreglo menu_items de esta colección
        */
       $lookup: {
         from: "menu_items",
         localField: "menu_items",
         foreignField: "_id",
         as: "menu_items_info",
       },
     },
     {
       /**
        * Ocultar los campos que no vamos a usar
        */
       $project: {
         _id: 0,
         name: 0,
         address: 0,
         contact: 0,
         manager: 0,
         employees: 0,
         menu_items: 0,
       },
     },
     {
       /**
        * Expandir el arreglo resultante
        */
       $unwind: {
         path: "$menu_items_info",
       },
     },
     {
       /**
        * Filtrar los items por categoria y fecha
        */
       $match: {
         $and: [
           { "menu_items_info.category": "CATEGORIA" },
           {
             $or: [
               {
                 "menu_items_info.season_start": {
                   $exists: false,
                 },
               },
               {
                 $and: [
                   {
                     "menu_items_info.season_start": {
                       $lte: new Date(),
                     },
                   },
                   {
                     "menu_items_info.season_end": {
                       $gte: new Date(),
                     },
                   },
                 ],
               },
             ],
           },
         ],
       },
     },
   ]
   ````

   

4. Buscar un platillo por nombre en una sucursal.

   ````json
   [
     {
       $match: {
         name: "Morelia",
       },
     },
     {
       $lookup: {
         from: "menu_items",
         localField: "menu_items",
         foreignField: "_id",
         as: "menu_items_info",
       },
     },
     {
       $project: {
         _id: 0,
         name: 0,
         address: 0,
         contact: 0,
         manager: 0,
         employees: 0,
         menu_items: 0,
       },
     },
     {
       $unwind: {
         path: "$menu_items_info",
       },
     },
     {
       $match: {
         "menu_items_info.name": {
           $regex: "CADENA*",
           $options: "i",
         },
       },
     },
   ]
   ````


## 2. Cuentas

1. Listar todas las cuentas por fecha.

   ````json
   db.bills.find().sort({date: -1})
   ````
2. Listar todas las cuentas de un mesero por id.

   ````json
   db.bills.find({"waiter": ObjectId("ID")})).sort({date: -1})
   ````
3. Listar todas las cuentas de un mesero por nombre.

   Primero se busca el mesero con:

   ````json
   db.employees.find({
       position: "Mesero",
         name: {
           $regex: "NOMBRE*",
           $options: 'i'
         }
   })
   ````

   El usuario hace clic en el mesero y se obtiene su id.
4. Listar los 5 platillos más comprados.

   ````json
   [
     {
       /**
        * Hide unused fields
        */
       $project: {
         _id: 0,
         date: 0,
         reservation: 0,
         waiter: 0,
         total: 0,
         tip: 0,
         table: 0,
         subsidiary: 0,
       },
     },
     {
       /**
        * Unwind the menu_items ids array
        */
       $unwind: {
         path: "$orders",
         preserveNullAndEmptyArrays: false,
       },
     },
     {
       /**
        * Count the repetition of each id
        */
       $group: {
         _id: "$orders",
         count: { $sum: 1 },
       },
     },
     {
       /**
        * Sort by repeated, from max to min
        */
       $sort: {
         count: -1,
       },
     },
     {
       /**
        * Provide the number of documents to limit.
        */
       $limit: 5,
     },
     {
       /**
        * Get the information of the top items
        */
       $lookup: {
         from: "menu_items",
         localField: "_id",
         foreignField: "_id",
         as: "top_info",
       },
     },
     {
       /**
        * Hide the unused fields
        */
       $project: {
         _id: 0,
         count: 0,
       },
     },
     {
       /**
        * Unwind the items info
        */
       $unwind: {
         path: "$top_info",
         preserveNullAndEmptyArrays: false,
       },
     },
   ]
   ````

   
5. Listar todas las cuentas de una sucursal por id.

   ````json
   db.bills.find({"subsidiary": ObjectId("ID")}).sort({date: -1})
   ````
6. Listar todas las cuentas de una sucursal por nombre.

   Primero se busca la sucursal con:

   ````json
   db.subsidiaries.find({
       name: {
           $regex: "NOMBRE*",
           $options: 'i'
         }
   })
   ````

   El usuario hace clic en la sucursal y se obtiene su id.

## Sucursales

1. Listar todas las sucursales
2. Buscar sucursales por domicilio

## Empleados

1. Listar todos los empleados

   Ordenar:

   - Por nombre
   - Por fecha de contratación
   - Por posición
   - Activos

2. Listar todos los empleados de una sucursal

   Ordenar:

   - Por nombre
   - Por fecha de contratación
   - Por posición
   - Activos

3. Buscar empleado por nombre

4. Buscar empleado por correo

## Reservaciones

1. Listar las reservaciones actuales (ordenar por fecha)
2. Buscar reservaciones por nombre
3. Buscar reservaciones por teléfono
4. Listar las reservaciones pasadas

## Permisos

1. Buscar permisos por nombre