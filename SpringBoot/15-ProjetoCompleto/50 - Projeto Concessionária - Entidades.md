# 50 — Projeto Concessionária — Entidades e Migrations

tags: #springboot #projeto #concessionária #jpa #migrations
links: [[49 - Projeto Concessionária - Visão Geral]] | [[51 - Projeto Concessionária - Controllers e DTOs]] | [[🗺️ Mapa Principal]]

---

## Migration V1 — Schema completo

```sql
-- src/main/resources/db/migration/V1__create_schema_inicial.sql

CREATE TABLE usuarios (
    id          BIGSERIAL    PRIMARY KEY,
    nome        VARCHAR(150) NOT NULL,
    email       VARCHAR(200) NOT NULL UNIQUE,
    senha_hash  VARCHAR(255) NOT NULL,
    role        VARCHAR(20)  NOT NULL DEFAULT 'USER',
    ativo       BOOLEAN      NOT NULL DEFAULT TRUE,
    criado_em   TIMESTAMP    NOT NULL DEFAULT NOW()
);

CREATE TABLE marcas (
    id           BIGSERIAL    PRIMARY KEY,
    nome         VARCHAR(100) NOT NULL UNIQUE,
    pais_origem  VARCHAR(100),
    ativo        BOOLEAN      NOT NULL DEFAULT TRUE,
    criado_em    TIMESTAMP    NOT NULL DEFAULT NOW(),
    atualizado_em TIMESTAMP
);

CREATE TABLE carros (
    id            BIGSERIAL       PRIMARY KEY,
    marca_id      BIGINT          NOT NULL REFERENCES marcas(id),
    modelo        VARCHAR(150)    NOT NULL,
    ano           INTEGER         NOT NULL,
    preco         DECIMAL(12, 2)  NOT NULL,
    cor           VARCHAR(50)     NOT NULL,
    quilometragem INTEGER         NOT NULL DEFAULT 0,
    status        VARCHAR(20)     NOT NULL DEFAULT 'DISPONIVEL',
    criado_em     TIMESTAMP       NOT NULL DEFAULT NOW(),
    atualizado_em TIMESTAMP,
    CONSTRAINT chk_carros_ano     CHECK (ano BETWEEN 1900 AND 2100),
    CONSTRAINT chk_carros_preco   CHECK (preco > 0),
    CONSTRAINT chk_carros_km      CHECK (quilometragem >= 0)
);

CREATE TABLE clientes (
    id            BIGSERIAL    PRIMARY KEY,
    nome          VARCHAR(150) NOT NULL,
    email         VARCHAR(200) NOT NULL UNIQUE,
    telefone      VARCHAR(20)  NOT NULL,
    cpf           VARCHAR(14)  UNIQUE,
    ativo         BOOLEAN      NOT NULL DEFAULT TRUE,
    criado_em     TIMESTAMP    NOT NULL DEFAULT NOW(),
    atualizado_em TIMESTAMP
);

CREATE TABLE vendas (
    id              BIGSERIAL       PRIMARY KEY,
    carro_id        BIGINT          NOT NULL REFERENCES carros(id),
    cliente_id      BIGINT          NOT NULL REFERENCES clientes(id),
    valor_venda     DECIMAL(12, 2)  NOT NULL,
    data_venda      TIMESTAMP       NOT NULL DEFAULT NOW(),
    forma_pagamento VARCHAR(30)     NOT NULL,
    observacao      TEXT,
    criado_em       TIMESTAMP       NOT NULL DEFAULT NOW(),
    CONSTRAINT chk_vendas_valor CHECK (valor_venda > 0)
);

-- Índices
CREATE INDEX idx_carros_marca     ON carros(marca_id);
CREATE INDEX idx_carros_status    ON carros(status);
CREATE INDEX idx_carros_ano       ON carros(ano);
CREATE INDEX idx_vendas_carro     ON vendas(carro_id);
CREATE INDEX idx_vendas_cliente   ON vendas(cliente_id);
CREATE INDEX idx_vendas_data      ON vendas(data_venda DESC);
```

## Migration V2 — Usuário admin inicial

```sql
-- src/main/resources/db/migration/V2__insert_usuario_admin.sql
-- Senha: Admin@123 (hash BCrypt)
INSERT INTO usuarios (nome, email, senha_hash, role)
VALUES (
    'Administrador',
    'admin@concessionaria.com',
    '$2a$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi',
    'ADMIN'
);
```

---

## Entidade Marca

```java
package com.concessionaria.api.marca;

import jakarta.persistence.*;
import org.springframework.data.annotation.*;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "marcas")
@EntityListeners(AuditingEntityListener.class)
public class Marca {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true, length = 100)
    private String nome;

    @Column(name = "pais_origem", length = 100)
    private String paisOrigem;

    @Column(nullable = false)
    private boolean ativo = true;

    @CreatedDate
    @Column(name = "criado_em", nullable = false, updatable = false)
    private LocalDateTime criadoEm;

    @LastModifiedDate
    @Column(name = "atualizado_em")
    private LocalDateTime atualizadoEm;

    @OneToMany(mappedBy = "marca", fetch = FetchType.LAZY)
    private List<Carro> carros = new ArrayList<>();

    protected Marca() {}

    public Marca(String nome, String paisOrigem) {
        this.nome = nome;
        this.paisOrigem = paisOrigem;
    }

    public void atualizar(String nome, String paisOrigem) {
        this.nome = nome;
        this.paisOrigem = paisOrigem;
    }

    public void desativar() { this.ativo = false; }
    public void ativar()    { this.ativo = true; }

    public Long getId()              { return id; }
    public String getNome()          { return nome; }
    public String getPaisOrigem()    { return paisOrigem; }
    public boolean isAtivo()         { return ativo; }
    public LocalDateTime getCriadoEm()    { return criadoEm; }
    public LocalDateTime getAtualizadoEm(){ return atualizadoEm; }
}
```

---

## Entidade Carro

```java
package com.concessionaria.api.carro;

import com.concessionaria.api.marca.Marca;
import jakarta.persistence.*;
import org.springframework.data.annotation.*;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;
import java.math.BigDecimal;
import java.time.LocalDateTime;

@Entity
@Table(name = "carros")
@EntityListeners(AuditingEntityListener.class)
public class Carro {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "marca_id", nullable = false)
    private Marca marca;

    @Column(nullable = false, length = 150)
    private String modelo;

    @Column(nullable = false)
    private Integer ano;

    @Column(nullable = false, precision = 12, scale = 2)
    private BigDecimal preco;

    @Column(nullable = false, length = 50)
    private String cor;

    @Column(nullable = false)
    private Integer quilometragem = 0;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private StatusCarro status = StatusCarro.DISPONIVEL;

    @CreatedDate
    @Column(name = "criado_em", nullable = false, updatable = false)
    private LocalDateTime criadoEm;

    @LastModifiedDate
    @Column(name = "atualizado_em")
    private LocalDateTime atualizadoEm;

    protected Carro() {}

    public Carro(Marca marca, String modelo, Integer ano, BigDecimal preco,
                 String cor, Integer quilometragem) {
        this.marca = marca;
        this.modelo = modelo;
        this.ano = ano;
        this.preco = preco;
        this.cor = cor;
        this.quilometragem = quilometragem;
    }

    public void atualizar(String modelo, Integer ano, BigDecimal preco,
                          String cor, Integer quilometragem) {
        this.modelo = modelo;
        this.ano = ano;
        this.preco = preco;
        this.cor = cor;
        this.quilometragem = quilometragem;
    }

    public void marcarComoVendido() {
        if (this.status != StatusCarro.DISPONIVEL) {
            throw new com.concessionaria.api.exception.RegraDeNegocioException(
                "Carro não está disponível para venda. Status atual: " + this.status
            );
        }
        this.status = StatusCarro.VENDIDO;
    }

    public void marcarComoReservado() {
        if (this.status != StatusCarro.DISPONIVEL) {
            throw new com.concessionaria.api.exception.RegraDeNegocioException(
                "Carro não pode ser reservado. Status atual: " + this.status
            );
        }
        this.status = StatusCarro.RESERVADO;
    }

    public boolean isDisponivel() { return this.status == StatusCarro.DISPONIVEL; }

    // Getters
    public Long getId()            { return id; }
    public Marca getMarca()        { return marca; }
    public String getModelo()      { return modelo; }
    public Integer getAno()        { return ano; }
    public BigDecimal getPreco()   { return preco; }
    public String getCor()         { return cor; }
    public Integer getQuilometragem() { return quilometragem; }
    public StatusCarro getStatus() { return status; }
    public LocalDateTime getCriadoEm()     { return criadoEm; }
    public LocalDateTime getAtualizadoEm() { return atualizadoEm; }
}
```

```java
package com.concessionaria.api.carro;

public enum StatusCarro {
    DISPONIVEL,
    RESERVADO,
    VENDIDO
}
```

---

## Entidade Cliente

```java
package com.concessionaria.api.cliente;

import jakarta.persistence.*;
import org.springframework.data.annotation.*;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;
import java.time.LocalDateTime;

@Entity
@Table(name = "clientes")
@EntityListeners(AuditingEntityListener.class)
public class Cliente {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 150)
    private String nome;

    @Column(nullable = false, unique = true, length = 200)
    private String email;

    @Column(nullable = false, length = 20)
    private String telefone;

    @Column(length = 14, unique = true)
    private String cpf;

    @Column(nullable = false)
    private boolean ativo = true;

    @CreatedDate
    @Column(name = "criado_em", nullable = false, updatable = false)
    private LocalDateTime criadoEm;

    @LastModifiedDate
    @Column(name = "atualizado_em")
    private LocalDateTime atualizadoEm;

    protected Cliente() {}

    public Cliente(String nome, String email, String telefone, String cpf) {
        this.nome = nome;
        this.email = email;
        this.telefone = telefone;
        this.cpf = cpf;
    }

    public void atualizar(String nome, String telefone) {
        this.nome = nome;
        this.telefone = telefone;
    }

    public void desativar() { this.ativo = false; }

    public Long getId()           { return id; }
    public String getNome()       { return nome; }
    public String getEmail()      { return email; }
    public String getTelefone()   { return telefone; }
    public String getCpf()        { return cpf; }
    public boolean isAtivo()      { return ativo; }
    public LocalDateTime getCriadoEm()     { return criadoEm; }
    public LocalDateTime getAtualizadoEm() { return atualizadoEm; }
}
```

---

## Entidade Venda

```java
package com.concessionaria.api.venda;

import com.concessionaria.api.carro.Carro;
import com.concessionaria.api.cliente.Cliente;
import jakarta.persistence.*;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;
import java.math.BigDecimal;
import java.time.LocalDateTime;

@Entity
@Table(name = "vendas")
@EntityListeners(AuditingEntityListener.class)
public class Venda {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "carro_id", nullable = false)
    private Carro carro;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "cliente_id", nullable = false)
    private Cliente cliente;

    @Column(name = "valor_venda", nullable = false, precision = 12, scale = 2)
    private BigDecimal valorVenda;

    @Column(name = "data_venda", nullable = false)
    private LocalDateTime dataVenda;

    @Column(name = "forma_pagamento", nullable = false, length = 30)
    private String formaPagamento;

    @Column(columnDefinition = "TEXT")
    private String observacao;

    @CreatedDate
    @Column(name = "criado_em", nullable = false, updatable = false)
    private LocalDateTime criadoEm;

    protected Venda() {}

    public Venda(Carro carro, Cliente cliente, BigDecimal valorVenda,
                 String formaPagamento, String observacao) {
        this.carro = carro;
        this.cliente = cliente;
        this.valorVenda = valorVenda;
        this.formaPagamento = formaPagamento;
        this.observacao = observacao;
        this.dataVenda = LocalDateTime.now();
    }

    public Long getId()                 { return id; }
    public Carro getCarro()             { return carro; }
    public Cliente getCliente()         { return cliente; }
    public BigDecimal getValorVenda()   { return valorVenda; }
    public LocalDateTime getDataVenda() { return dataVenda; }
    public String getFormaPagamento()   { return formaPagamento; }
    public String getObservacao()       { return observacao; }
    public LocalDateTime getCriadoEm()  { return criadoEm; }
}
```

---

## Próximas notas
- [[51 - Projeto Concessionária - Controllers e DTOs]] — implementação completa das camadas
