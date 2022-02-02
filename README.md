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
- Exemplo violando o princípio OCP — Open-Closed Principle:

Veja o exemplo de violação na classe DebitoConta, que contém o método responsável por realizar as debitos das contas de Conta Corrente e Conta Poupança através de um Enum chamado “TipoConta”. Como podemos ver no código abaixo, o problema dessa implementação está na complexidade. Quanto mais regras forem sendo criadas, mais cases vão existir e a manutenção dessa classe vai ficar inviável. Além disso, o acoplamento da classe DebitoConta vai aumentar, porque vai cada vez mais depender de mais classes.
```
namespace SOLID.OCP.Violacao
{
    public class DebitoConta
    {
        public void Debitar(decimal valor, string conta, TipoConta tipoConta)
        {
            if (tipoConta == TipoConta.Corrente)
            {
                // Debita Conta Corrente
            }

            if (tipoConta == TipoConta.Poupanca)
            {
                // Valida Aniversário da Conta
                // Debita Conta Poupança
            }
        }
    }
}
```
- Exemplo resolvendo a violação do princípio OCP — Open-Closed Principle:

Primeiramente colocaremos a classe DebitoConta com o tipo abstrato e tendo as propriedades (NumeroTransacao, Debitar, FormatarTransacao). De acordo com o código abaixo:

```
using System;
using System.Linq;

namespace SOLID.OCP.Solucao
{
    public abstract class DebitoConta
    {
        public string NumeroTransacao { get; set; }
        public abstract string Debitar(decimal valor, string conta);

        public string FormatarTransacao()
        {
            const string chars = "ABCasDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
            var random = new Random();
            NumeroTransacao = new string(Enumerable.Repeat(chars, 15)
              .Select(s => s[random.Next(s.Length)]).ToArray());

            // Numero de transacao formatado
            return NumeroTransacao;
        } 
    }
}
````
Criaremos classes como DebitoContaCorrente e DebitoPoupança que irão herdar as características da classe pai que será DebitoConta. De acordo com código abaixo:
```
namespace SOLID.OCP.Solucao
{
    public class DebitoContaCorrente : DebitoConta
    {
        public override string Debitar(decimal valor, string conta)
        {
            // Debita Conta Corrente
            return FormatarTransacao();
        }
    }
}

namespace SOLID.OCP.Solucao
{
    public class DebitoContaPoupanca : DebitoConta
    {
        public override string Debitar(decimal valor, string conta)
        {
            // Valida Aniversário da Conta
            // Debita Conta Corrente
            return FormatarTransacao();
        }
    }
}
```
Com isso, conseguimos enxugar a classe DebitoConta e fazer com que ela não precise conhecer o comportamento das diversas contas. Ou seja, vamos fechar as classes DebitoContaCorrente e DebitoPoupança para mudanças e caso outras regras surjam para serem utilizadas na classe DebitoConta.
Open-Closed Principle também é base para o padrão de projeto Strategy, estamos obedecendo o OCP.

## L – Liskov Substitution Principle (Princípio da substituição de Liskov)

- As classes derivadas devem ser substituíveis pelas suas classes bases.
- Suponhamos que você tenha uma classe Pessoa. Essa classe contém atributos como nome, CPF, RG… Mas, e se você criar uma outra classe chamada Aluno, quais os atributos que a classe Aluno pode conter? Se você pensou em nome, CPF, RG, você está certíssimo, porém, não é interessante criar esses mesmos atributos para a classe aluno, o ideal seria que Aluno herdasse de Pessoa.
- Esse é o princípio que traz a ideia de herança. Temos uma classe pai, que geralmente possui atributos genéricos e temos uma classe filha, que herda os atributos da classe pai e pode ter outros atributos específicos para si mesma. No nosso exemplo, a classe Aluno poderia herdar todos os atributos da classe Pessoa e ter também outros atributos como nota, presença.
- Exemplo violando o princípio L – Liskov Substitution Principle:
- Sobrescrever/implementar um método que não faz nada;
- Lançar uma exceção inesperada;
- Retornar valores de tipos diferentes da classe base.

Segue o código abaixo:
```
namespace SOLID.LSP.Violacao
{
    public class Retangulo
    {
        public virtual double Altura { get; set; }
        public virtual double Largura { get; set; }
        public double Area { get { return Altura * Largura; } }
    }
}

namespace SOLID.LSP.Violacao
{
    public class Quadrado : Retangulo
    {
        public override double Altura
        {
            set { base.Altura = base.Largura = value; }
        }

        public override double Largura
        {
            set { base.Altura = base.Largura = value; }
        }
    }
}

using System;

namespace SOLID.LSP.Violacao
{
    public class CalculoArea
    {
        private static void ObterAreaRetangulo(Retangulo ret)
        {
            Console.Clear();
            Console.WriteLine("Calculo da área do Retangulo");
            Console.WriteLine();
            Console.WriteLine(ret.Altura + " * " + ret.Largura);
            Console.WriteLine();
            Console.WriteLine(ret.Area);
            Console.ReadKey();
        }

        public static void Calcular()
        {
            var quad = new Quadrado()
            {
                Altura = 10,
                Largura = 5
            };

            ObterAreaRetangulo(quad);
        }
    }
}
```
- Resolvendo a violação do princípio L – Liskov Substitution Principle:

```
namespace SOLID.LSP.Solucao
{
    public class Retangulo : Paralelogramo
    {
        public Retangulo(int altura, int largura)
            :base(altura,largura)
        {

        }
    }
}

using System;

namespace SOLID.LSP.Solucao
{
    public class Quadrado : Paralelogramo
    {
        public Quadrado(int altura, int largura)
            : base(altura, largura)
        {
            if(largura != altura)
                throw new ArgumentException("Os dois lados do quadrado precisam ser iguais");
        }
    }
}

namespace SOLID.LSP.Solucao
{
    public abstract class Paralelogramo
    {
        protected Paralelogramo(int altura, int largura)
        {
            Altura = altura;
            Largura = largura;
        }

        public double Altura { get; private set; }
        public double Largura { get; private set ; }
        public double Area { get { return Altura * Largura; } } 
    }
}

using System;

namespace SOLID.LSP.Solucao
{
    public class CalculoArea
    {
        private static void ObterAreaParalelogramo(Paralelogramo ret)
        {
            Console.Clear();
            Console.WriteLine("Calculo da área do Retangulo");
            Console.WriteLine();
            Console.WriteLine(ret.Altura + " * " + ret.Largura);
            Console.WriteLine();
            Console.WriteLine(ret.Area);
            Console.ReadKey();
        }

        public static void Calcular()
        {
            var quad = new Quadrado(5,5);
            var ret = new Retangulo(10, 5);

            ObterAreaParalelogramo(quad);
            ObterAreaParalelogramo(ret);
        }
    }
}
```
Para não violar o Liskov Substitution Principle, além de estruturar muito bem as suas abstrações, em alguns casos, você precisara usar a injeção de dependência e também usar outros princípios do SOLID. O princípio de Liskov Substitution Principle nos permite usar o polimorfismo com mais confiança. Podemos chamar nossas classes derivadas referindo-se à sua classe base sem preocupações com resultados inesperados.

## I – Interface Segregation Principle (Princípio da Segregação da Interface)

- Classes não devem ser forçadas a depender de métodos que não usam. 
- Quando você aplica o princípio de herança, fazendo uma classe herdar da outra, sua classe filha é obrigada a implementar os métodos da classe pai e como você já deve estar imaginando, isso vai contra os princípios do SOLID, pois não é nada interessante que uma classe implementa métodos que não é útil para ela. 
- Com o princípio de segregação de interface, é possível implementar somente o que importa para as nossas classes. Interfaces que possuem muitos comportamentos são difíceis de manter e evoluir, e devem ser evitadas.
- Exemplo violando o princípio ISP — Interface Segregation Principle:

Em um cenário fictício para criação de programa de e commerce, verificamos que as classes CadastroProduto e CadastroCliente herdam a mesma interface que possui mesmos métodos, porém metódo EnviarEmail() não corresponde para a classe CadastroProduto, pois produto não tem e-mail. Dessa forma, estamos violando o Interface Segregation Principle e o LSP também.
Segue o código abaixo:
```
namespace SOLID.ISP.Violacao
{
    public interface ICadastro
    {
        void ValidarDados();
        void SalvarBanco();
        void EnviarEmail();
    }
}

using System;

namespace SOLID.ISP.Violacao
{
    public class CadastroProduto : ICadastro
    {
        public void ValidarDados()
        {
            // Validar valor
        }

        public void SalvarBanco()
        {
            // Insert tabela Produto
        }

        public void EnviarEmail()
        {
            // Produto não tem e-mail, o que eu faço agora???
            throw new NotImplementedException("Esse metodo não serve pra nada");
        }
    }
}

namespace SOLID.ISP.Violacao
{
    public class CadastroCliente : ICadastro
    {
        public void ValidarDados()
        {
            // Validar CPF, Email
        }

        public void SalvarBanco()
        {
            // Insert na tabela Cliente
        }

        public void EnviarEmail()
        {
            // Enviar e-mail para o cliente
        }
    }
}
```
- Resolvendo a violação do princípio ISP — Interface Segregation Principle:

Resolveremos esse problema criando as interfaces mais específicas, segue o código abaixo:
```
using SOLID.ISP.Solucao.Interfaces;

namespace SOLID.ISP.Solucao
{
    public class CadastroProduto : ICadastroProduto
    {
        public void ValidarDados()
        {
            // Validar valor
        }

        public void SalvarBanco()
        {
            // Insert tabela Produto
        }
    }
}

using SOLID.ISP.Solucao.Interfaces;

namespace SOLID.ISP.Solucao
{
    public class CadastroCliente : ICadastroCliente
    {
        public void ValidarDados()
        {
            // Validar CPF, Email
        }

        public void SalvarBanco()
        {
            // Insert na tabela Cliente
        }

        public void EnviarEmail()
        {
            // Enviar e-mail para o cliente
        }
    }
}

namespace SOLID.ISP.Solucao.Interfaces
{
    public interface ICadastroProduto : ICadastro
    {
        void ValidarDados();
    }
}

namespace SOLID.ISP.Solucao.Interfaces
{
    public interface ICadastroCliente : ICadastro
    {
        void ValidarDados();
        void EnviarEmail();
    }
}

namespace SOLID.ISP.Solucao.Interfaces
{
    public interface ICadastro
    {
        void SalvarBanco();
    }
}
```
## D – Dependency Inversion Principle (Princípio da inversão da dependência)

- O princípio da inversão de dependência traz a ideia de que: 
- Módulos de alto nível não devem depender de módulos de baixo nível. Ambos devem depender da abstração.
- Abstrações não devem depender de detalhes. Os detalhes devem depender das abstrações. E isso se dá porque abstrações mudam menos e facilitam a mudança de comportamento e as futuras evoluções do código.
- Em outras palavras, os módulos que são classes de alto nível devem depender de conceitos, também chamadas de abstrações independente de como funcionam, ou seja, a função da inversão de dependência faz com que os softwares se desassociem dos módulos. 


## Os princípios SOLID devem ser aplicados para se obter os benefícios da orientação a objetos, tais como:

- Clean Code
- DDD
- SOLID
- YAGNI, KISS, DRY
- Gang of Four design patterns (Creational, Structural e Behavioral)
