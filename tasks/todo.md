# Task: Crear skill para especificaciones con interrogacion profunda

## Plan
- [x] Revisar formato y convenciones de skills existentes
- [x] Definir nombre, triggers y alcance de la nueva skill
- [x] Crear `specification-interrogator/SKILL.md` con frontmatter e instrucciones operativas
- [x] Validar estructura de la skill con checklist rapido
- [x] Documentar resultado final

## Acceptance criteria
- Existe una carpeta de skill en kebab-case
- Existe `SKILL.md` con frontmatter valido (`name` y `description`)
- La skill instruye a preguntar de forma exhaustiva antes de planear
- La skill incluye la instruccion explicita de iniciar preguntando que se quiere construir

## Review
- Se creo `specification-interrogator/SKILL.md` con frontmatter valido y nombre kebab-case.
- La skill incluye el prompt base solicitado y define el gate de "no planear hasta despejar unknowns".
- Se incluyo inicio obligatorio con "What do you want to build?" y alternativa en espanol.
- Se documentaron formato de salida, ejemplos de trigger y troubleshooting.
