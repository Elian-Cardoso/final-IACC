using System;
using ZoologicoPOO.Models;
using ZoologicoPOO.Interfaces;
using System.Collections.Generic;

List<Animal> animais = new();
List<Funcionario> funcionarios = new();

bool executando = true;

while (executando)
{
    Console.WriteLine("\n--- Sistema do Zoológico ---");
    Console.WriteLine("1. Cadastrar Animal");
    Console.WriteLine("2. Cadastrar Funcionário");
    Console.WriteLine("3. Interagir Animal-Funcionário");
    Console.WriteLine("4. Sair");
    Console.Write("Escolha uma opção: ");
    string opcao = Console.ReadLine();

    Console.WriteLine();

    switch (opcao)
    {
        case "1":
            Console.Write("Tipo de animal (Mamifero/Ave/Reptil): ");
            string tipoAnimal = Console.ReadLine().ToLower();

            Console.Write("Nome: ");
            string nomeAnimal = Console.ReadLine();

            Console.Write("Idade: ");
            int idadeAnimal = int.Parse(Console.ReadLine());

            Console.Write("Peso: ");
            double pesoAnimal = double.Parse(Console.ReadLine());

            Console.Write("Espécie: ");
            string especieAnimal = Console.ReadLine();

            Animal animal = tipoAnimal switch
            {
                "mamifero" => new Mamifero(nomeAnimal, idadeAnimal, pesoAnimal, especieAnimal),
                "ave" => new Ave(nomeAnimal, idadeAnimal, pesoAnimal, especieAnimal),
                "reptil" => new Reptil(nomeAnimal, idadeAnimal, pesoAnimal, especieAnimal),
                _ => null
            };

            if (animal != null)
            {
                animais.Add(animal);
                Console.WriteLine($"Animal cadastrado com sucesso: Nome: {animal.Nome}, Idade: {animal.Idade}, Peso: {animal.Peso}kg, Espécie: {animal.Especie}.");
            }
            else
            {
                Console.WriteLine("Tipo de animal inválido.");
            }
            break;

        case "2":
            Console.Write("Tipo de funcionário (Veterinario/Zelador): ");
            string tipoFunc = Console.ReadLine().ToLower();

            Console.Write("Nome: ");
            string nomeFunc = Console.ReadLine();

            Console.Write("Idade: ");
            int idadeFunc = int.Parse(Console.ReadLine());

            Funcionario funcionario = tipoFunc switch
            {
                "veterinario" => new Veterinario(nomeFunc, idadeFunc),
                "zelador" => new Zelador(nomeFunc, idadeFunc),
                _ => null
            };

            if (funcionario != null)
            {
                funcionarios.Add(funcionario);
                Console.WriteLine($"Funcionário cadastrado com sucesso: Nome: {funcionario.Nome}, Idade: {funcionario.Idade}, Cargo: {funcionario.Cargo}.");
            }
            else
            {
                Console.WriteLine("Tipo de funcionário inválido.");
            }
            break;

        case "3":
            if (animais.Count == 0 || funcionarios.Count == 0)
            {
                Console.WriteLine("Cadastre pelo menos um animal e um funcionário antes de interagir.");
                break;
            }

            Console.WriteLine("\nAnimais disponíveis:");
            for (int i = 0; i < animais.Count; i++)
            {
                Console.WriteLine($"{i} - {animais[i].Nome} ({animais[i].Especie})");
            }

            Console.Write("Selecione o índice do animal: ");
            int indexAnimal = int.Parse(Console.ReadLine());
            Animal animalSel = animais[indexAnimal];

            Console.WriteLine("\nFuncionários disponíveis:");
            for (int i = 0; i < funcionarios.Count; i++)
            {
                Console.WriteLine($"{i} - {funcionarios[i].Nome} ({funcionarios[i].Cargo})");
            }

            Console.Write("Selecione o índice do funcionário: ");
            int indexFunc = int.Parse(Console.ReadLine());
            Funcionario funcSel = funcionarios[indexFunc];

            Console.WriteLine();

            if (funcSel is Veterinario vet)
            {
                vet.ConsultarAnimal(animalSel);
                vet.RealizarTratamento(animalSel);
            }
            else if (funcSel is Zelador zel)
            {
                zel.AlimentarAnimal(animalSel);
                zel.CuidarHabitat(animalSel);
            }
            break;

        case "4":
            executando = false;
            Console.WriteLine("Encerrando o sistema...");
            break;

        default:
            Console.WriteLine("Opção inválida.");
            break;
    }
}
