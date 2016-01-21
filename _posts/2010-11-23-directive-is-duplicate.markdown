---
layout: page
title:  nginx&#58; directive is duplicate
date:   2010-11-23 00:00:00
---

Заканчиваю разработку модуля для nginx’а под названием ustats (“upstream statistics”), показывающего статистику по бэкендам,
которая нашему отделу необходима для мониторинга активности серверов. В процессе разработки столкнулся с интересной проблемой,
которая, с одной стороны, была вызвана моим пренебрежением к мануалам, а с другой – неполнотой этих мануалов
(речь о [Emiller’s Guide To Nginx Module Development](http://www.evanmiller.org/nginx-modules-guide.html) и прочих).

<!--break-->

Итак, пусть наш модуль ModuleName должен поддерживать конфигурационную директиву `test_dir`, которая может
иметь целое неотрицательное значение. Контекст не важен, но пусть для простоты это `location`.

Реализация поддержки директивы выглядела бы примерно так. Объявляется структура, хранящая настройки модуля:

{% highlight cpp %}
typedef struct
{
...
    ngx_uint_t test_dir;
...
} ngx_http_modulename_loc_conf_t;
{% endhighlight %}

В список команд модуля добавляется описание директивы:

{% highlight cpp %}
static ngx_command_t  ngx_http_modulename_commands[] =
{
...
    {
        ngx_string("test_dir"),
        NGX_HTTP_LOC_CONF | NGX_CONF_TAKE1,
        ngx_conf_set_num_slot,
        NGX_HTTP_LOC_CONF_OFFSET,
        offsetof(ngx_http_modulename_loc_conf_t, test_dir),
        NULL
    },
...
{% endhighlight %}

Если в структуре контекста модуля не прописан указатель на функцию создания конфига модуля, прописываем:

{% highlight cpp %}
static ngx_http_module_t  ngx_http_modulename_module_ctx =
{
...
    ngx_http_modulename_create_loc_conf,       /* create location configuration */
    /* merging function pointer here */
...
};
{% endhighlight %}

И последний шаг – сама функция создания конфигурации модуля:

{% highlight cpp %}
static void * ngx_http_modulename_create_loc_conf(ngx_conf_t *cf)
{
    ngx_http_modulename_loc_conf_t  *conf;

    conf = ngx_pcalloc(cf->pool, sizeof(ngx_http_modulename_loc_conf_t));
    if (conf == NULL)
        return NGX_CONF_ERROR;

    conf->test_dir = 10;

    return conf;
}
{% endhighlight %}

Я действительно думал, что присвоив переменной `test_dir` значение 10 я тем самым устанавливаю её значение по-умолчанию.
Вот после такой реализации я и огрёб ту самую интересную проблему. В настройках nginx’а было прописано

{% highlight cpp %}
...
location /modulename {
   test_dir 50;
}
...
{% endhighlight %}

на что он и начал ругаться, что

{% highlight cpp %}
[emerg]: "test_dir" directive is duplicate in /opt/nginx/conf/nginx.conf:173
{% endhighlight %}

Эта ошибка возникла не потому, что директива действительно случайно была продублирована в своём контексте. **Причина состояла в том,
что в функции создания конфигурации модуля нужно было указать, что переменной не присвоено никакого значения**, а именно:

{% highlight cpp %}
static void * ngx_http_modulename_create_loc_conf(ngx_conf_t *cf)
{
    ngx_http_modulename_loc_conf_t  *conf;
    conf = ngx_pcalloc(cf->pool, sizeof(ngx_http_modulename_loc_conf_t));
    if (conf == NULL)
        return NGX_CONF_ERROR;
    conf->test_dir = NGX_CONF_UNSET_UINT;
    return conf;
}
{% endhighlight %}

После такого исправления всё работает.

**UPDATE**

Установка дефолтных значений директив производится внутри функции слияния конфигов. Указатель на соответствующую функцию должен быть прописал в контексте модуля:
{% highlight cpp %}
static ngx_http_module_t  ngx_http_modulename_module_ctx =
{
...
    ngx_http_modulename_create_loc_conf,       /* create location configuration */
    ngx_http_modulename_merge_loc_conf        /* merge location configuration */
};
{% endhighlight %}

Сама функция выглядит так:

{% highlight cpp %}
static char* ngx_http_modulename_merge_loc_conf(ngx_conf_t *cf, void *parent, void *child)
{
    ngx_http_modulename_loc_conf_t *prev = parent;
    ngx_http_modulename_loc_conf_t *conf = child;

    ngx_conf_merge_uint_value(conf->test_dir, prev->test_dir, 70 /* дефолтное значение для директивы */);

    // ...

    return NGX_CONF_OK;
}
{% endhighlight %}