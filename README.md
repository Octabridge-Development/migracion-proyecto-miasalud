V7-a: Diagnosticar 1,065 contactos sin estado
Esta consulta cruza los contactos nulos con las tablas de staging para ver si el estado es recuperable.

SQL
SELECT c.id, scs.state, scc.last_state
FROM miasaludnatural_test.contacts c
JOIN miasaludnatural_test.external_ids e ON e.contact_id = c.id
LEFT JOIN miasaludnatural_test.staging_contact_state_azure scs ON scs.contact_id = c.id
LEFT JOIN miasaludnatural_test.staging_campaign_contact_azure scc ON scc.contact_id = c.id
WHERE e.external_system = 'manychat' AND c.current_state_id IS NULL;
V7-b: Verificación post-corrección de estados
Después de hacer los UPDATE, esta consulta debe devolver 0 contactos sin estado.

SQL
SELECT COUNT(*) 
FROM miasaludnatural_test.contacts c
JOIN miasaludnatural_test.external_ids e ON e.contact_id = c.id
WHERE e.external_system = 'manychat' AND c.current_state_id IS NULL;


Tarea V3 y V4 (Asesores y WhatsApp)
V3-a: Segmentar 7,243 sin asesor comercial
Esta es la que estábamos viendo para sacar el CSV. Agrupa los nulos por estado del CRM.

SQL
SELECT cs.name, COUNT(*) 
FROM miasaludnatural_test.contacts c
JOIN miasaludnatural_test.external_ids e ON e.contact_id = c.id
LEFT JOIN miasaludnatural_test.contact_assignments ca ON ca.contact_id = c.id AND ca.is_active = true
LEFT JOIN miasaludnatural_test.crm_states cs ON cs.id = c.current_state_id
WHERE e.external_system = 'manychat' AND ca.commercial_advisor_id IS NULL
GROUP BY cs.name 
ORDER BY COUNT(*) DESC;
V3-b: Verificación post-corrección de asignaciones faltantes
El manual indica que para verificar la V3-b se debe re-ejecutar la query V3-a original. El conteo de contactos en estados avanzados ("Requiere Acción") debería bajar a 0.

V4: Resolver 43 WhatsApp sin teléfono
Verifica si quedan contactos asociados al canal de WhatsApp que tengan su campo de teléfono encriptado nulo. El resultado esperado es 0.

SQL
SELECT COUNT(*) 
FROM miasaludnatural_test.contacts c
JOIN miasaludnatural_test.channels ch ON ch.id = c.channel_id
WHERE lower(ch.name) LIKE '%whatsapp%' AND c.phone_encrypted IS NULL;


Tarea O-08 (Adaptación de Funciones)
O-08: Verificar funciones almacenadas en el nuevo esquema
Esta consulta lista todas las rutinas (funciones) para asegurar que se hayan creado correctamente dentro del esquema de prueba y no en public.

SQL
SELECT routine_name 
FROM information_schema.routines
WHERE routine_schema = 'miasaludnatural_test'
ORDER BY routine_name;
