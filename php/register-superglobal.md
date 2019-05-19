# PHP 如何处理 query string 中的数组

> 请求 domain/path?a[]=1&a[index]=2

```php
    <?php

    var_dump($_GET);

    // array(1) { ["a"]=> array(2) { [0]=> string(1) "1" ["index"]=> string(1) "2" } }
```

## 初始化变量 PHP 源码函数调用

- php_request_startup (main.c)
- php_hash_environment
- treat_data(php_register_variable_safe)
- php_regisger_variable_ex (php_variables.c)

对于 query string 、form-data 、x-www-form-urlencoded 数组类型的处理，相关代码 php_register_variable_ex in php_variables.c

```c
    PHPAPI void php_register_variable_ex(char *var_name, zval *val, zval *track_vars_array)
    {
        char *p = NULL;
        char *ip = NULL;        /* index pointer */
        char *index;
        char *var, *var_orig;
        size_t var_len, index_len;
        zval gpc_element, *gpc_element_p;
        zend_bool is_array = 0;
        HashTable *symtable1 = NULL;
        ALLOCA_FLAG(use_heap)

        assert(var_name != NULL);

        if (track_vars_array && Z_TYPE_P(track_vars_array) == IS_ARRAY) {
            symtable1 = Z_ARRVAL_P(track_vars_array);
        }

        if (!symtable1) {
            /* Nothing to do */
            zval_ptr_dtor_nogc(val);
            return;
        }


        /* ignore leading spaces in the variable name */
        while (*var_name==' ') {
            var_name++;
        }

        /*
         * Prepare variable name
         */
        var_len = strlen(var_name);
        var = var_orig = do_alloca(var_len + 1, use_heap);
        memcpy(var_orig, var_name, var_len + 1);

        /* ensure that we don't have spaces or dots in the variable name (not binary safe) */
        for (p = var; *p; p++) {
            if (*p == ' ' || *p == '.') {
                *p='_';
            } else if (*p == '[') { // 通过检查参数名称中时候有 '[' 来判断参数是否时数组类型
                is_array = 1;
                ip = p;
                *p = 0;
                break;
            }
        }

        // ...中间省略

        if (is_array) {
            int nest_level = 0;
            while (1) {
                char *index_s;
                size_t new_idx_len = 0;

                if(++nest_level > PG(max_input_nesting_level)) {
                    HashTable *ht;
                    /* too many levels of nesting */

                    if (track_vars_array) {
                        ht = Z_ARRVAL_P(track_vars_array);
                        zend_symtable_str_del(ht, var, var_len);
                    }

                    zval_ptr_dtor_nogc(val);

                    /* do not output the error message to the screen,
                     this helps us to to avoid "information disclosure" */
                    if (!PG(display_errors)) {
                        php_error_docref(NULL, E_WARNING, "Input variable nesting level exceeded " ZEND_LONG_FMT ". To increase the limit change max_input_nesting_level in php.ini.", PG(max_input_nesting_level));
                    }
                    free_alloca(var_orig, use_heap);
                    return;
                }

                ip++;
                index_s = ip;
                // 确认是否确实为数组类型，并确定关联数组的 `index`
                if (isspace(*ip)) {
                    ip++;
                }
                if (*ip==']') { // 如果参数名称为 `a[]` 则表明参数没有指明 `index`
                    index_s = NULL;
                } else {
                    ip = strchr(ip, ']');
                    if (!ip) {
                        /* PHP variables cannot contain '[' in their names, so we replace the character with a '_' */
                        *(index_s - 1) = '_';

                        index_len = 0;
                        if (index) {
                            index_len = strlen(index);
                        }
                        goto plain_var;
                        return;
                    }
                    *ip = 0;
                    new_idx_len = strlen(index_s);
                }

    // ...下面代码省略
    }
```

PHP 源码处理超全局变量的函数

```c
    // php_variables.c
    void php_startup_auto_globals(void)
    {
        zend_register_auto_global(zend_string_init_interned("_GET", sizeof("_GET")-1, 1), 0, php_auto_globals_create_get);
        zend_register_auto_global(zend_string_init_interned("_POST", sizeof("_POST")-1, 1), 0, php_auto_globals_create_post);
        zend_register_auto_global(zend_string_init_interned("_COOKIE", sizeof("_COOKIE")-1, 1), 0, php_auto_globals_create_cookie);
        zend_register_auto_global(zend_string_init_interned("_SERVER", sizeof("_SERVER")-1, 1), PG(auto_globals_jit), php_auto_globals_create_server);
        zend_register_auto_global(zend_string_init_interned("_ENV", sizeof("_ENV")-1, 1), PG(auto_globals_jit), php_auto_globals_create_env);
        zend_register_auto_global(zend_string_init_interned("_REQUEST", sizeof("_REQUEST")-1, 1), PG(auto_globals_jit), php_auto_globals_create_request);
        zend_register_auto_global(zend_string_init_interned("_FILES", sizeof("_FILES")-1, 1), 0, php_auto_globals_create_files);
    }
```
