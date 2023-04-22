---
layout: default
title: libsde-utils
---

Набор библиотек, реализующих вспомогательные функции для программ SDE:
  * libsde-utils - резолвер путей, вспомогательные функции для запуска программ, работы со строками, и вывода отладочного лога
  * libsde-utils-gtk - вспомогательные функции для работы с графическим тулкитом gtk.
  * libsde-utils-x11 - вспомогательные функции для работы с оконной системой X11.
  * libsde-utils-jansson - вспомогательные функции для работы jansson, библиотекой для парсинга JSON.

## Резолвер путей

Библиотека libsde-utils реализует резолвер путей к ресурсам приложения. Полные пути таким образом не вкомпилированы в приложение, а определяются в ран-тайме. Это позволяет создавать портабельные приложения, а также подменять/добавлять ресурсы per-user без перекомпиляции приложения.

Вот так резолвинг ресурсов выглядит в логе запуска waterline:

```
[DBG] 2023-04-22 18:23:56 [<unknown>] pointer 0x558C58C65650 identified as module waterline
[DBG] 2023-04-22 18:23:56 [<unknown>] agent waterline registered with default prefix /usr
[DBG] 2023-04-22 18:23:56 [waterline] resource waterline:locale resolved as "/usr/share/locale"
[DBG] 2023-04-22 18:23:56 [waterline] resource waterline:waterline/images resolved as "/usr/share/waterline/images"
[DBG] 2023-04-22 18:23:56 [waterline] config sde/waterline:t5:config:USER resolved as "/home/vadim/.config/sde/waterline/t5/config"
```

А вот так он выполняется в коде. Сначала определяется идентификатор агента. В рамках одного процесса может находиться произвольное число агентов:

```
/* global variable */
gchar * _wtl_agent_id = NULL;

/* code in main() */
_wtl_agent_id = su_path_resolve_agent_id_by_pointer(main, "waterline");
su_path_register_default_agent_prefix(_wtl_agent_id, PACKAGE_INSTALLATION_PREFIX);
```

Затем выполняется поиск ресурса для заданного агента:

```
gchar * locale_dir = su_path_resolve_resource(_wtl_agent_id, "locale", NULL);
```

В настоящее время резолвер путей не реализует весь объем запланированных возможностей по гибкой перестройке путей программ. По мере доработки резолвера эта статья будет дополняться.
