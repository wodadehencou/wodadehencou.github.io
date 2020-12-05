---
layout: post
title: test ec_points_mul
date: 2020-12-05 23:03
tags: []
category:
---

```c

#include "openssl/bn.h"
#include "openssl/ec.h"
#include "openssl/evp.h"
#include "openssl/obj_mac.h"
#include "sys/time.h"
#include <math.h>
#include <openssl/crypto.h>
#include <openssl/ossl_typ.h>
#include <stdio.h>
#include <string.h>
#include <time.h>

#ifdef GM
#define NID_SM2 NID_sm2p256v1
#else
#define NID_SM2 NID_sm2
#endif

static double gettimedouble(void) {
    struct timeval tv;
    gettimeofday(&tv, NULL);
    return tv.tv_usec * 0.000001 + tv.tv_sec;
}

void print_number(double x) {
    double y = x;
    int c = 0;
    if (y < 0.0) {
        y = -y;
    }
    while (y > 0 && y < 100.0) {
        y *= 10.0;
        c++;
    }
    printf("%.*f", c, x);
}

void randomize(unsigned char *a, int len) {
    int i;
    for (i = 0; i < len; i++) {
        a[i] = rand();
    }
}

void ec_test(EC_GROUP *group) {
    double begin, end, total, total_cycles;
    total_cycles = 10000.0;
    double b;

    EC_POINT *r_point;
    BIGNUM *p_value;
    BIGNUM *x;
    BIGNUM *y;
    //   BN_CTX *ctx = BN_CTX_new();
    BN_CTX *ctx = NULL;

    p_value = BN_new();
    x = BN_new();
    y = BN_new();
    r_point = EC_POINT_new(group);

    BIGNUM *order;
    order = BN_new();
    EC_GROUP_get_order(group, order, ctx);
    BN_rand_range(p_value, order);
    EC_POINT_mul(group, r_point, p_value, NULL, NULL, ctx);
    EC_POINT_get_affine_coordinates_GFp(group, r_point, x, y, ctx);
    // EC_POINT_make_affine(group, r_point, ctx);
    // EC_POINT_point2bn(group, r_point, POINT_CONVERSION_UNCOMPRESSED, x, ctx);

    printf("group %d curve test:\n", EC_GROUP_get_curve_name(group));
    begin = gettimedouble();
    for (b = 0; b < total_cycles; b++) {
        // EC_POINT_point2bn(group, r_point, POINT_CONVERSION_UNCOMPRESSED, x, ctx);
        EC_POINT_mul(group, r_point, NULL, r_point, x, ctx);
        EC_POINT_get_affine_coordinates_GFp(group, r_point, x, y, ctx);
        // EC_POINT_make_affine(group, r_point, ctx);
    }
    total = gettimedouble() - begin;
    printf("point_mul:");
    print_number(1e6 * (total / total_cycles));
    printf("us\n");

    begin = gettimedouble();
    for (b = 0; b < total_cycles; b++) {
        // EC_POINT_point2bn(group, r_point, POINT_CONVERSION_UNCOMPRESSED, x, ctx);
        EC_POINT_mul(group, r_point, x, NULL, NULL, ctx);
        EC_POINT_get_affine_coordinates_GFp(group, r_point, x, y, ctx);
        // EC_POINT_make_affine(group, r_point, ctx);
    }
    total = gettimedouble() - begin;
    printf("base_point_mul:");
    print_number(1e6 * (total / total_cycles));
    printf("us\n");

    begin = gettimedouble();
    for (b = 0; b < total_cycles; b++) {
        // EC_POINT_point2bn(group, r_point, POINT_CONVERSION_UNCOMPRESSED, x, ctx);
        EC_POINT_mul(group, r_point, x, r_point, y, ctx);
        EC_POINT_get_affine_coordinates_GFp(group, r_point, x, y, ctx);
        // EC_POINT_make_affine(group, r_point, ctx);
    }
    total = gettimedouble() - begin;
    printf("aG+bH:");
    print_number(1e6 * (total / total_cycles));
    printf("us\n");

    EC_POINT_free(r_point);
    BN_free(p_value);
    BN_free(x);
    BN_free(y);
    BN_free(order);
    BN_CTX_free(ctx);
}

void ecdsa_test(EC_GROUP *group) {
    printf("group %d ecdsa test:\n", EC_GROUP_get_curve_name(group));

    unsigned char text[] = "test ecdsa signature";
    const unsigned char *message = &text[0];

    EC_KEY *eckey = EC_KEY_new();
    EC_KEY_set_group(eckey, group);
    EC_KEY_generate_key(eckey);
    ECDSA_SIG *signature =
        ECDSA_do_sign(message, strlen((const char *)message), eckey);
    ECDSA_do_verify(message, strlen((const char *)message), signature, eckey);

    double begin, end, total;
	int total_cycles = 10000;
    double b;

    begin = gettimedouble();
    for (b = 0; b < total_cycles; b++) {
        ECDSA_do_sign(message, strlen((const char *)message), eckey);
    }
    total = gettimedouble() - begin;
    printf("ecdsa sign:");
    print_number((1e6 * total) / (double)total_cycles);
    printf("us; ");
    print_number((1.0 / total) * (double)total_cycles);
    printf("t/s\n");

    begin = gettimedouble();
    for (b = 0; b < total_cycles; b++) {
        ECDSA_do_verify(message, strlen((const char *)message), signature, eckey);
    }
    total = gettimedouble() - begin;
    printf("ecdsa verify:");
    print_number((1e6 * total) / (double)total_cycles);
    printf("us; ");
    print_number((1.0 / total) * (double)total_cycles);
    printf("t/s\n");

    ECDSA_SIG_free(signature);
    EC_KEY_free(eckey);
}

void hash_test(const EVP_MD *type) {
    printf("HASH %d test: \n", EVP_MD_type(type));
    unsigned char *message_1k = malloc(1024 * sizeof(unsigned char));
    randomize(message_1k, 1024);
    unsigned char *message_8k = malloc(8192 * sizeof(unsigned char));
    randomize(message_8k, 8192);
    double begin, end, total;
    int total_cycles = 10000;
    double b;

    EVP_MD_CTX *context = EVP_MD_CTX_new();

    begin = gettimedouble();
    for (b = 0; b < total_cycles; b++) {
        EVP_DigestInit_ex(context, type, NULL);
        EVP_DigestUpdate(context, message_1k, 1024);
        unsigned char hash[EVP_MAX_MD_SIZE];
        unsigned int lengthOfHash = 0;

        EVP_DigestFinal_ex(context, hash, &lengthOfHash);
    }
    total = gettimedouble() - begin;
    printf("hash 1kBytes:");
    print_number((1e6 * total) / (double)total_cycles);
    printf("us; ");
    print_number((((double)total_cycles * 1024) / total) / (1024 * 1024));
    printf("MB/s \n");

    begin = gettimedouble();
    for (b = 0; b < total_cycles; b++) {
        EVP_DigestInit_ex(context, type, NULL);
        EVP_DigestUpdate(context, message_8k, 8192);
        unsigned char hash[EVP_MAX_MD_SIZE];
        unsigned int lengthOfHash = 0;

        EVP_DigestFinal_ex(context, hash, &lengthOfHash);
    }
    total = gettimedouble() - begin;
    printf("hash 8kBytes:");
    print_number((1e6 * total) / (double)total_cycles);
    printf("us; ");
    print_number((((double)total_cycles * 8192) / total) / (1024 * 1024));
    printf("MB/s \n");

    EVP_MD_CTX_free(context);
}

void symmetric_test(const EVP_CIPHER *cipher) {
    printf("Symmetric Cipher %d test: \n", EVP_CIPHER_type(cipher));
    unsigned char *message_1k = malloc(1024 * sizeof(unsigned char));
    randomize(message_1k, 1024);
    unsigned char *message_8k = malloc(8192 * sizeof(unsigned char));
    randomize(message_8k, 1024);
    double begin, end, total;
    int total_cycles = 10000;
    double b;

    EVP_CIPHER_CTX *context = EVP_CIPHER_CTX_new();
    unsigned char *key = malloc(16);
    randomize(key, 16);
    unsigned char *iv = malloc(EVP_CIPHER_block_size(cipher));
    randomize(iv, EVP_CIPHER_block_size(cipher));

    unsigned char *plaintext = malloc(10240 * sizeof(unsigned char));
    unsigned char *ciphertext = malloc(10240 * sizeof(unsigned char));
    int len;
    int plaintext_len;
    int ciphertext_len;
    // encrypt
    EVP_EncryptInit_ex(context, cipher, NULL, key, iv);
    EVP_EncryptUpdate(context, ciphertext, &len, message_1k, 1024);
    ciphertext_len = len;
    EVP_EncryptFinal_ex(context, ciphertext + len, &len);
    ciphertext_len += len;
    // decrypt
    EVP_DecryptInit_ex(context, cipher, NULL, key, iv);
    EVP_DecryptUpdate(context, plaintext, &len, ciphertext, ciphertext_len);
    plaintext_len = len;
    EVP_DecryptFinal_ex(context, plaintext + len, &len);
    plaintext_len += len;

    if (plaintext_len != 1024) {
        printf("cipher self test fail, length not equal \n");
        return;
    }
    int i;
    for (i = 0; i < 1024; i++) {
        if (plaintext[i] != message_1k[i]) {
            printf("cipher self test fail, %d not equal \n", i);
            return;
        }
    }

    begin = gettimedouble();
    for (b = 0; b < total_cycles; b++) {
        EVP_EncryptInit_ex(context, cipher, NULL, key, iv);
        EVP_EncryptUpdate(context, ciphertext, &len, message_1k, 1024);
        ciphertext_len = len;
        EVP_EncryptFinal_ex(context, ciphertext + len, &len);
        ciphertext_len += len;
    }
    total = gettimedouble() - begin;
    printf("symmetric 1kBytes encrypt:");
    print_number((1e6 * total) / (double)total_cycles);
    printf("us; ");
    print_number((((double)total_cycles * 1024) / total) / (1024 * 1024));
    printf("MB/s \n");

    begin = gettimedouble();
    for (b = 0; b < total_cycles; b++) {
        EVP_DecryptInit_ex(context, cipher, NULL, key, iv);
        EVP_DecryptUpdate(context, plaintext, &len, ciphertext, ciphertext_len);
        plaintext_len = len;
        EVP_DecryptFinal_ex(context, plaintext + len, &len);
        plaintext_len += len;
    }
    total = gettimedouble() - begin;
    printf("symmetric 1kBytes decrypt:");
    print_number((1e6 * total) / (double)total_cycles);
    printf("us; ");
    print_number((((double)total_cycles * 1024) / total) / (1024 * 1024));
    printf("MB/s \n");

    begin = gettimedouble();
    for (b = 0; b < total_cycles; b++) {
        EVP_EncryptInit_ex(context, cipher, NULL, key, iv);
        EVP_EncryptUpdate(context, ciphertext, &len, message_8k, 8192);
        ciphertext_len = len;
        EVP_EncryptFinal_ex(context, ciphertext + len, &len);
        ciphertext_len += len;
    }
    total = gettimedouble() - begin;
    printf("symmetric 8kBytes encrypt:");
    print_number((1e6 * total) / (double)total_cycles);
    printf("us; ");
    print_number((((double)total_cycles * 8192) / total) / (1024 * 1024));
    printf("MB/s \n");

    begin = gettimedouble();
    for (b = 0; b < total_cycles; b++) {
        EVP_DecryptInit_ex(context, cipher, NULL, key, iv);
        EVP_DecryptUpdate(context, plaintext, &len, ciphertext, ciphertext_len);
        plaintext_len = len;
        EVP_DecryptFinal_ex(context, plaintext + len, &len);
        plaintext_len += len;
    }
    total = gettimedouble() - begin;
    printf("symmetric 8kBytes decrypt:");
    print_number((1e6 * total) / (double)total_cycles);
    printf("us; ");
    print_number((((double)total_cycles * 8192) / total) / (1024 * 1024));
    printf("MB/s \n");

    EVP_CIPHER_CTX_free(context);
}

void ec_points_mul_test(EC_GROUP *group, int n) {
    double begin, end, total, total_cycles;
    total_cycles = 10000.0;
    double b;

    EC_POINT *r_point;
    BIGNUM *p_value;
    BIGNUM *x;
    BIGNUM *y;
    //   BN_CTX *ctx = BN_CTX_new();
    BN_CTX *ctx = NULL;

    p_value = BN_new();
    x = BN_new();
    y = BN_new();
    r_point = EC_POINT_new(group);

    BIGNUM *order;
    order = BN_new();
    EC_GROUP_get_order(group, order, ctx);
    BN_rand_range(p_value, order);
    EC_POINT_mul(group, r_point, p_value, NULL, NULL, ctx);
    EC_POINT_get_affine_coordinates_GFp(group, r_point, x, y, ctx);
    // EC_POINT_make_affine(group, r_point, ctx);
    // EC_POINT_point2bn(group, r_point, POINT_CONVERSION_UNCOMPRESSED, x, ctx);

	EC_POINT ** ps;
	BIGNUM ** ks;
	ps = OPENSSL_malloc(n*sizeof(EC_POINT*));
	ks = OPENSSL_malloc(n*sizeof(BIGNUM*));
	int i;
	for (i=0; i<n; i++) {
		ks[i] = BN_new();
		ps[i] = EC_POINT_new(group);
		BN_rand_range(ks[i], order);
		EC_POINT_mul(group, ps[i], ks[i], NULL, NULL, ctx);
        EC_POINT_get_affine_coordinates_GFp(group, ps[i], x, y, ctx);
	}
    begin = gettimedouble();
    for (b = 0; b < total_cycles; b++) {
        // EC_POINT_point2bn(group, r_point, POINT_CONVERSION_UNCOMPRESSED, x, ctx);
        EC_POINTs_mul(group, r_point, NULL, n, (const EC_POINT **)ps, (const BIGNUM **)ks, ctx);
        EC_POINT_get_affine_coordinates_GFp(group, r_point, x, y, ctx);
        // EC_POINT_make_affine(group, r_point, ctx);
    }
    total = gettimedouble() - begin;
    printf("%d points_mul(ki*Pi+...):", n);
    print_number(1e6 * (total / total_cycles));
    printf("us\n");

	for (i=0; i<n; i++) {
		EC_POINT_free(ps[i]);
		BN_free(ks[i]);
	}
	OPENSSL_free(ps);
	OPENSSL_free(ks);
    EC_POINT_free(r_point);
    BN_free(p_value);
    BN_free(x);
    BN_free(y);
    BN_free(order);
    BN_CTX_free(ctx);
}

int main() {

    EC_GROUP *group;
    printf("====================================\n");
    printf("P256 test:\n");
    group = EC_GROUP_new_by_curve_name(NID_X9_62_prime256v1);
	ec_test(group);
	int i;
	for (i =1; i<10; i++) {
		ec_points_mul_test(group, i);
	}
#if 0
    ecdsa_test(group);

#ifndef NO_SM
    printf("====================================\n");
    printf("SM2 test:\n");
    group = EC_GROUP_new_by_curve_name(NID_SM2);
    ec_test(group);
    ecdsa_test(group);
    EC_GROUP_free(group);
#endif

    printf("====================================\n");
    printf("SHA256 test:\n");
    hash_test(EVP_sha256());

#ifndef NO_SM
    printf("====================================\n");
    printf("SM3 test:\n");
    hash_test(EVP_sm3());
#endif

    printf("====================================\n");
    printf("AES ECB test:\n");
	symmetric_test(EVP_aes_128_ecb());

    printf("====================================\n");
    printf("AES CBC test:\n");
    symmetric_test(EVP_aes_128_cbc());

#ifndef NO_SM
    printf("====================================\n");
    printf("SM4 ECB test:\n");
    symmetric_test(EVP_sm4_ecb());

    printf("====================================\n");
    printf("SM4 CBC test:\n");
    symmetric_test(EVP_sm4_cbc());
#endif
#endif


    EC_GROUP_free(group);
    return 0;
}


```
