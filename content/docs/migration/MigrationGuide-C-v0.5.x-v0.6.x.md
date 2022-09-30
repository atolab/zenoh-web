---
title: "Migrating from Zenoh-C v0.5.x zenoh-net API to Zenoh-C v0.6.x zenoh API"
weight : 5400
menu:
  docs:
    parent: migration
---

### Opening a session

All types and operations from the `zn_*` primitives have been updated and migrated to the `z_*` primitives.

*zenoh v0.5.x*
```C
zn_properties_t *config = zn_config_default();
zn_session_t *s = zn_open(config);
if (s == NULL) {
    printf("Unable to open session!\n");
    exit(-1);
}
```

*zenoh v0.6.x*
```C
z_owned_config_t config = z_config_default();
z_owned_session_t s = z_open(z_move(config));
if (!z_check(s)) {
    printf("Unable to open session!\n");
    exit(-1);
}
```

### Subscribing

For this release, Zenoh-C only supports subscribers with callbacks. It is possible to access samples through a callback by calling the `callback` function passed as argument on `declare_subscriber` function.

When declaring a subscriber, keyexpr optimizations
(i.e., keyexpr declaration) will be automatically performed if required.

Finer configuration is performed with the help of an options struct.

*zenoh-net v0.5.x*
```C
void data_handler(const zn_sample_t *sample, const void *arg) {
    printf(">> [Subscription listener] Received (%.*s, %.*s)\n",
        sample->key.len, sample->key.val,
        sample->value.len, sample->value.val);
}

// (...)

zn_subscriber_t *sub = zn_declare_subscriber(s, zn_rname("/key/expression"), zn_subinfo_default(), data_handler, NULL);
if (sub == NULL) {
    printf("Unable to declare subscriber.\n");
    exit(-1);
}
```

*zenoh v0.6.x*
```C
void data_handler(const z_sample_t *sample, void *arg)
{
    char *keystr = z_keyexpr_to_string(sample->keyexpr);
    printf(">> [Subscriber] Received ('%s': '%.*s')\n",
           keystr, (int)sample->payload.len, sample->payload.start);
    free(keystr);
}

// (...)

z_owned_closure_sample_t callback = z_closure(data_handler);
z_owned_subscriber_t sub = z_declare_subscriber(z_loan(s), z_keyexpr("key/expression"), z_move(callback), NULL);
if (!z_check(sub))
{
    printf("Unable to declare subscriber.\n");
    exit(-1);
}
```

### Publishing

The `write` operation has been replaced by a `put` operation.

When declaring a publisher, keyexpr optimizations (i.e., keyexpr declaration) will be automatically performed if required.

Finer configuration is performed with the help of an options struct.

*zenoh-net v0.5.x*
```C
zn_write(s, reskey, "value", strlen("value"));
```
 
*zenoh v0.6.x*
```C
if (z_put(z_loan(s), z_loan(keyexpr), "value", strlen("value"), NULL) < 0) {
    printf("Put has failed!\n");
}
```

The `write_ext` operation has been removed. Configuration is now performed with the help of a option struct.

*zenoh-net v0.5.x*
```C
zn_write_ext(s, reskey, "value", strlen("value"), Z_ENCODING_TEXT_PLAIN, Z_DATA_KIND_DEFAULT, zn_congestion_control_t_BLOCK);
```

*zenoh v0.6.x*
```C
z_put_options_t options = z_put_options_default();
options.congestion_control = Z_CONGESTION_CONTROL_DROP;
options.encoding = Z_ENCODING_PREFIX_TEXT_PLAIN;
options.priority = Z_PRIORITY_DATA;
if (z_put(z_loan(s), z_loan(keyexpr), "value", strlen("value"), &options) < 0) {
    printf("Put has failed!\n");
}
```

The `declare_publisher` now returns a publisher object upon which `put` and `delete` operations can be performed.

*zenoh-net v0.5.x*
```C
zn_publisher_t *pub = zn_declare_publisher(s, reskey);
if (pub == NULL) {
    printf("Unable to declare publisher.\n");
    exit(-1);
}
```

*zenoh v0.6.x*
```C
z_owned_publisher_t pub = z_declare_publisher(z_loan(s), z_keyexpr("key/expression"), NULL);
if (!z_check(pub)) {
    printf("Unable to declare publisher!\n");
    exit(-1);
}
z_publisher_put(z_loan(pub), "value", strlen("value"), NULL);
```

### Querying

The `query_collect` operation has been replaced by a `get` operation. The `get` operation is no longer blocking and returning a list of replies, but instead it makes replies accessible through a callback by calling the `callback` function passed as argument on `get` function.

When declaring a publisher, keyexpr optimizations (i.e., keyexpr declaration) will be automatically performed if required.

Finer configuration is performed with the help of an options struct.

*zenoh-net v0.5.x*
```C
void reply_handler(const zn_source_info_t *info, const zn_sample_t *sample, const void *arg) {
    printf(">> [Reply handler] received (%.*s, %.*s)\n",
        sample->key.len, sample->key.val,
        sample->value.len, sample->value.val);
}

// (...)

zn_query(s, zn_rname("/key/expression"), "", zn_query_target_default(), zn_query_consolidation_default(), reply_handler, NULL);
```

*zenoh v0.6.x*
```C
void reply_dropper(void *ctx)
{
    printf(">> Received query final notification\n");
}

void reply_handler(z_owned_reply_t *oreply, void *ctx)
{
    if (z_reply_is_ok(oreply)) {
        z_sample_t sample = z_reply_ok(oreply);
        char *keystr = z_keyexpr_to_string(sample.keyexpr);
        printf(">> Received ('%s': '%.*s')\n", keystr, (int)sample.payload.len, sample.payload.start);
        free(keystr);
    } else {
        printf(">> Received an error\n");
    }
}

// (...)

z_get_options_t opts = z_get_options_default();
opts.target = Z_QUERY_TARGET_ALL;
z_owned_closure_reply_t callback = z_closure(reply_handler, reply_dropper);
if (z_get(z_loan(s), z_keyexpr("key/expression"), "", z_move(callback), &opts) < 0) {
    printf("Unable to send query.\n");
    exit(-1);
}
```

### Queryable

It is possible to access queries through a callback by calling the `callback` function passed as argument on `declare_queryable` function.

When declaring a queryable, keyexpr optimizations (i.e., keyexpr declaration) will be automatically performed if required.

Finer configuration is performed with the help of an options struct.

The `send_reply` operation has been also extended with an options struct for finer configuration.

*zenoh-net v0.5.x*
```C
void query_handler(zn_query_t *query, const void *ctx)
{
    z_string_t res = zn_query_res_name(query);
    z_string_t pred = zn_query_predicate(query);
    printf(">> [Query handler] Handling '%.*s?%.*s'\n", (int)res.len, res.val, (int)pred.len, pred.val);
    zn_send_reply(query, "/key/expression", "value", strlen("value"));
}

// (...)

zn_queryable_t *qable = zn_declare_queryable(s, zn_rname("/key/expression"), ZN_QUERYABLE_EVAL, query_handler, NULL);
if (qable == NULL) {
    printf("Unable to declare queryable.\n");
    exit(-1);
}
```

*zenoh v0.6.x*
```C
void query_handler(const z_query_t *query, void *ctx)
{
    char *keystr = z_keyexpr_to_string(z_query_keyexpr(query));
    z_bytes_t pred = z_query_value_selector(query);
    printf(">> [Queryable ] Received Query '%s%.*s'\n", keystr, (int)pred.len, pred.start);
    z_query_reply(query, z_keyexpr("key/expression"), "value", strlen("value"), NULL);
    free(keystr);
}

// (...)

z_owned_closure_query_t callback = z_closure_query(query_handler);
z_owned_queryable_t qable = z_declare_queryable(z_loan(s), z_keyexpr("key/expression"), z_closure(callback), NULL);
if (!z_check(qable)) {
    printf("Unable to create queryable.\n");
    exit(-1);
}
```


### Examples

More examples are available here: 

[*zenoh v0.6.0*](https://github.com/eclipse-zenoh/zenoh-c/tree/master/examples)

[*zenoh-net v0.5.0-beta9*](https://github.com/eclipse-zenoh/zenoh-c/tree/0.5.0-beta.9)
