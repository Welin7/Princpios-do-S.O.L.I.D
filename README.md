# Principios do S.O.L.I.D
O SOLID é um acrônimo criado por Michael Feathers, após observar que cinco princípios da orientação a objetos e design de código — Criados por Robert C. Martin (a.k.a. Uncle Bob) e abordados no artigo The Principles of OOD — poderiam se encaixar nesta palavra.
O Solid possui cinco princípios considerados como boas práticas no desenvolvimento de software que ajudam os programadores a escrever os códigos mais limpos, separando as responsabilidades, diminuindo acoplamentos, facilitando na refatoração e estimulando o reaproveitamento do código. Os princípios são :

## S – Single Responsibility Principle (Princípio da responsabilidade única)
- Esse primeiro princípio diz que “uma classe deve ter apenas um motivo para mudar”, ou seja, deve ter uma única responsabilidade.
- O princípio da responsabilidade única enfatiza que uma classe deve ter apenas um objetivo, ou seja, ela deve possuir apenas uma função ou funções similares.
- Fazer com que uma única classe execute funções que tem haver com aquela classe.
- Exemplo violando o princípio SRP — Single Responsibility Principle:

Veja o exemplo de violação na classe Cliente, que contém as propriedades de um cliente e um método chamado AdicionarCliente(). Como podemos ver no código abaixo, não existe uma única responsabilidade nessa classe que está violando o princípio SRP.
```
using System;
using System.Data;
using System.Data.SqlClient;
using System.Net.Mail;

namespace SOLID.SRP.Violacao
{
    public class Cliente
    {
        public int ClienteId { get; set; }
        public string Nome { get; set; }
        public string Email { get; set; }
        public string CPF { get; set; }
        public DateTime DataCadastro { get; set; }

        public string AdicionarCliente()
        {
            if (!Email.Contains("@"))
                return "Cliente com e-mail inválido";

            if (CPF.Length != 11)
                return "Cliente com CPF inválido";


            using (var cn = new SqlConnection())
            {
                var cmd = new SqlCommand();

                cn.ConnectionString = "MinhaConnectionString";
                cmd.Connection = cn;
                cmd.CommandType = CommandType.Text;
                cmd.CommandText = "INSERT INTO CLIENTE (NOME, EMAIL CPF, DATACADASTRO) VALUES (@nome, @email, @cpf, @dataCad))";

                cmd.Parameters.AddWithValue("nome", Nome);
                cmd.Parameters.AddWithValue("email", Email);
                cmd.Parameters.AddWithValue("cpf", CPF);
                cmd.Parameters.AddWithValue("dataCad", DataCadastro);

                cn.Open();
                cmd.ExecuteNonQuery();
            }

            var mail = new MailMessage("empresa@empresa.com", Email);
            var client = new SmtpClient
            {
                Port = 25,
                DeliveryMethod = SmtpDeliveryMethod.Network,
                UseDefaultCredentials = false,
                Host = "smtp.google.com"
            };

            mail.Subject = "Bem Vindo.";
            mail.Body = "Parabéns! Você está cadastrado.";
            client.Send(mail);

            return "Cliente cadastrado com sucesso!";
        }
    }
}
```
Sendo assim, não há necessidade dessa classe ter duas responsabilidades que não seria a função dela de ter uma lógica de adicionar um novo cliente.
Com isso estamos violando esse principio gerando alguns problemas em seu código, sendo eles:
- ***Falta de coesão — uma classe não deve assumir responsabilidades que não são suas;***
- ***Alto acoplamento — Mais responsabilidades geram um maior nível de dependências, deixando o sistema engessado e frágil para alterações;***
- ***Dificuldades na implementação de testes automatizados — É difícil de “mockar” esse tipo de classe;***
- ***Dificuldades para reaproveitar o código.***

- Exemplo resolvendo a violação do princípio SRP — Single Responsibility Principle:

A classe “Cliente” deve ser especializada em um único assunto e possuir apenas uma responsabilidade dentro do software, ou seja, a classe deve ter uma única tarefa ou ação para executar. A classe cliente terá somente as suas propriedades, segue o código abaixo:
```
using System;

namespace SOLID.SRP.Solucao
{
    public class Cliente
    {
        public int ClienteId { get; set; }
        public string Nome { get; set; }
        public Email Email { get; set; }
        public Cpf Cpf { get; set; }
        public DateTime DataCadastro { get; set; }

        public bool Validar()
        {
            return Email.Validar() && Cpf.Validar();
        }
    }
}
```
Já na parte de conexão com a base dados, criamos um pattern repository de acordo com código abaixo:

```
using System.Data;
using System.Data.SqlClient;

namespace SOLID.SRP.Solucao
{
    public class ClienteRepository
    {
        public void AdicionarCliente(Cliente cliente)
        {
            using (var cn = new SqlConnection())
            {
                var cmd = new SqlCommand();

                cn.ConnectionString = "MinhaConnectionString";
                cmd.Connection = cn;
                cmd.CommandType = CommandType.Text;
                cmd.CommandText = "INSERT INTO CLIENTE (NOME, EMAIL CPF, DATACADASTRO) VALUES (@nome, @email, @cpf, @dataCad))";

                cmd.Parameters.AddWithValue("nome", cliente.Nome);
                cmd.Parameters.AddWithValue("email", cliente.Email);
                cmd.Parameters.AddWithValue("cpf", cliente.Cpf);
                cmd.Parameters.AddWithValue("dataCad", cliente.DataCadastro);

                cn.Open();
                cmd.ExecuteNonQuery();
            }
        }
    }
}
```
Já no método AdicionarCliente(), criamos uma classe de serviço do cliente que será responsável por todas ações de inserção do novo cliente.

```
namespace SOLID.SRP.Solucao
{
    public class ClienteService
    {
        public string AdicionarCliente(Cliente cliente)
        {
            if (!cliente.Validar())
                return "Dados inválidos";

            var repo = new ClienteRepository();
            repo.AdicionarCliente(cliente);

            EmailServices.Enviar("empresa@empresa.com", cliente.Email.Endereco, "Bem Vindo", "Parabéns está Cadastrado");

            return "Cliente cadastrado com sucesso";
        }
    }
}
```
## O – Open-Closed Principle (Princípio Aberto-Fechado)

- O princípio aberto-fechado traz a ideia de que as classes da aplicação devem ser abertas para extensões e fechadas para modificações, ou seja, outras classes podem ter acesso ao que aquela classe possui, porém, não podem alterá-las.
- Isso porque alterar uma classe pai pode ser perigoso, já que outras classes dentro da aplicação podem estar utilizando-a. Ao realizar uma alteração nela, impactará todas as outras que estão utilizando ela.
- Talvez você esteja pensando: “e se a minha classe precisar executar uma outra tarefa?” Você pode simplesmente criar uma nova tarefa dentro da classe. A ideia aqui não é não mexer na classe em hipótese alguma, e sim, caso necessário, adicionar uma nova função àquela classe e não alterar o que já existe nela.


## Parte 3

- Visão geral sobre ORMs
- [Material de apoio](material_apoio/material_apoio_parte_3.md)
- CRUD com EF (Cadastro diretores e filmes)
- Linq (To SQL)

## Parte 4

- Repository Pattern 
- AutoMapper, DTOs 
- Dependence Injection 
- Fluent Validator 
- JWT
- Debbugar VSCode

## Parte 5

- Visão geral sobre arquitetura 
- Monolithic vs Microservices
- N-Tier
- Multi-Tenancy

## Parte 6

- Clean Code
- DDD
- SOLID
- YAGNI, KISS, DRY
- Gang of Four design patterns (Creational, Structural e Behavioral)
