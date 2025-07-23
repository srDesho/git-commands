# Git: How to Change Author Email in Commit History

## 📋 Escenario del Problema
Tienes commits en tu historial de Git con una dirección de correo incorrecta (`myemail@example.com`) y necesitas cambiarla por la correcta (`cristianlai775@gmail.com`) mientras preservas toda la demás información de los commits.

## 🎯 Métodos de Solución

### Método 1: Corrección de un Solo Commit (Arreglo Rápido)
Si solo necesitas arreglar el **último commit**:
```bash
git commit --amend --author="Tu Nombre <correo-correcto@dominio.com>" --no-edit
```

### Método 2: Múltiples Commits Recientes (Rebase Interactivo)
Para arreglar **múltiples commits recientes**:

```bash
# 1. Identifica cuántos commits necesitas arreglar
git log --oneline --format="%h | %an | %ae"

# 2. Inicia rebase interactivo para los últimos N commits
git rebase -i HEAD~4

# 3. Cambia 'pick' por 'edit' para los commits a modificar
# En el editor que se abre, cambia:
# edit <hash-del-commit> Mensaje del commit 1
# edit <hash-del-commit> Mensaje del commit 2

# 4. Para cada commit marcado, Git pausará. Luego ejecuta:
git commit --amend --author="Tu Nombre <correo-correcto@dominio.com>" --no-edit
git rebase --continue

# 5. Repite para cada commit
```

### Método 3: Corrección Masiva con git filter-branch
Para cambiar el correo **en todo el historial**:

```bash
git filter-branch --env-filter '
if [ "$GIT_AUTHOR_EMAIL" = "correo-viejo@example.com" ]; then
    GIT_AUTHOR_EMAIL="correo-nuevo@dominio.com"
    GIT_AUTHOR_NAME="Tu Nombre"
    GIT_COMMITTER_EMAIL="$GIT_AUTHOR_EMAIL"
    GIT_COMMITTER_NAME="$GIT_AUTHOR_NAME"
fi
' --tag-name-filter cat -- --all
```

### Método 4: Usando git filter-repo (Enfoque Moderno)
**Instala primero:** `pip install git-filter-repo`

```bash
# Crea un archivo mailmap
echo "<correo-viejo@example.com> Tu Nombre <correo-nuevo@dominio.com>" > mailmap.txt

# Aplica las correcciones
git filter-repo --mailmap mailmap.txt --force
```

## 🔧 Pasos de Verificación
Después de hacer los cambios:

```bash
# Verifica todos los autores/emails únicos
git log --format="%an <%ae>" | sort | uniq

# Verifica que no queden correos viejos
git log --format="%ae" | grep "correo-viejo" || echo "No se encontraron correos viejos ✓"

# Vista del historial corregido
git log --oneline -10 --format="%h | %an | %ae | %s"
```

## 🚀 Push Final
```bash
# Push forzado para actualizar el repositorio remoto
git push origin master --force

# Si usas la rama main
git push origin main --force
```

## ⚠️ Advertencias Importantes
1. **Haz backup primero**: `git branch backup-antes-de-cambiar-email`
2. **Comunica** con los miembros del equipo antes de hacer push forzado
3. **Los demás necesitarán** re-clonar o hacer `git pull --rebase`
4. **Configura globalmente** para prevenir problemas futuros:
   ```bash
   git config --global user.email "correo-correcto@dominio.com"
   git config --global user.name "Tu Nombre"
   ```

## 📝 Ejemplo Real
```bash
# Original: 4 commits con correo incorrecto
git log --oneline -4 --format="%h | %an | %ae"
# 82b8f1e | Cristian Montaño Laime | myemail@example.com
# d323c4d | Cristian Montaño Laime | myemail@example.com
# 064315d | Cristian Montaño Laime | myemail@example.com
# adc60c2 | Cristian Montaño Laime | myemail@example.com

# Corrige con rebase
git rebase -i HEAD~4
# Marca todos como 'edit', luego para cada uno:
git commit --amend --author="Cristian Montaño Laime <cristianlai775@gmail.com>" --no-edit
git rebase --continue

# Verifica la corrección
git log --oneline -4 --format="%h | %an | %ae"
# Todos ahora muestran: cristianlai775@gmail.com
```

## 🎉 Resultado
- ✅ Historial de commits actualizado con el correo correcto
- ✅ Gráfico de contribuciones de GitHub coloreado correctamente
- ✅ Todas las fechas y mensajes de commits preservados
- ✅ Integridad del repositorio mantenida

---

*Esta guía documenta el proceso real usado para corregir el correo del autor en el historial de un repositorio Git.*