# DZ-2

## Задание №2
Разработать инструмент командной строки для визуализации графа
зависимостей, включая транзитивные зависимости. Сторонние средства для
получения зависимостей использовать нельзя.
Зависимости определяются для git-репозитория. Для описания графа
зависимостей используется представление Graphviz. Визуализатор должен
выводить результат на экран в виде графического изображения графа.
Построить граф зависимостей для коммитов, в узлах которого содержатся
сообщения.
Ключами командной строки задаются:
• Путь к программе для визуализации графов.
• Путь к анализируемому репозиторию.
Все функции визуализатора зависимостей должны быть покрыты тестами.

## Содержание проекта


git_dep.py - Код программы

test_git_dep - Тесты программы

commit_dependecies.png - составленный граф

commit-tree-example.zip - клонированный репозиторий

## Код программы 
import json
import os
from graphviz import Digraph
import requests

# Load configuration
def load_config(config_file='config.json'):
    """Загружает конфигурационный файл.""" 
    with open(config_file, 'r') as file:
        return json.load(file)

# Fetch dependencies
def fetch_dependencies(package_name, repository_url):
    """Загружает зависимости для указанного пакета."""
    package_id = package_name.split(',')[0].strip()
    print(f"Fetching dependencies from: {repository_url}/{package_id.lower()}/index.json")
    
    response = requests.get(f"{repository_url}/{package_id.lower()}/index.json")
    if response.status_code != 200:
        print(f"Error fetching dependencies for package {package_name}, status code: {response.status_code}")
        exit(1)
    
    data = response.json()
    available_versions = [entry['catalogEntry']['version'] for entry in data['items'][0]['items']]
    print(f"Available versions: {available_versions}")
    
    version = package_name.split('Version=')[1].strip() if 'Version=' in package_name else None
    if version and version not in available_versions:
        print(f"Specified version {version} not found for package {package_name}.")
        exit(1)

    selected_version = version if version else available_versions[-1]
    print(f"Selected version: {selected_version}")
    
    dependencies = []
    for entry in data['items'][0]['items']:
        if entry['catalogEntry']['version'] == selected_version:
            dependencies = entry['catalogEntry'].get('dependencyGroups', [])
            break

    if not dependencies:
        print(f"No dependencies found for {package_name}, version: {selected_version}")
        exit(1)

    print(f"Found dependencies for version {selected_version}")
    return dependencies

# Create dependency graph
def create_graph(dependencies, package_name, output_path):
    """Создает и сохраняет граф зависимостей в формате dot."""
    graph = Digraph(comment=f'Dependency graph for {package_name}')
    for group in dependencies:
        target_framework = group.get('targetFramework', 'Unknown Framework')
        graph.node(target_framework, shape='box')
        for dependency in group.get('dependencies', []):
            dep_id = dependency['id']
            graph.node(dep_id)
            graph.edge(target_framework, dep_id)

    # Сохраняем граф в файл
    try:
        print(f"Attempting to save graph to: {output_path}")
        graph.save(output_path)
        print(f"Dependency graph saved to {output_path}.")
    except Exception as e:
        print(f"Error saving graph: {e}")

# Основной процесс
def main():
    # Загружаем конфигурацию
    config = load_config()
    
    # Получаем зависимости
    dependencies = fetch_dependencies(config['package_name'], config['repository_url'])

    # Создаем и сохраняем граф
    create_graph(dependencies, config['package_name'], config['output_path'])

    # Визуализируем граф
    os.system(f"{config['graph_tool_path']} -Tpng {config['output_path']} -o {config['output_path'].replace('.dot', '.png')}")
    os.system(f"open {config['output_path'].replace('.dot', '.png')}")  # Открыть граф (для macOS)
    
if __name__ == "__main__":
    main()


## Запуск программы

Предварительно нужно склонировать репозиторий используемый для анализа

``` git clone https://github.com/jason4pok/commit-tree-example ```

Программа запускается из командной строки

``` python git_dep.py "D:\conf2\git_dependency_visualizer\commit-tree-example" "C:\Program Files (x86)\Graphviz\bin\dot.exe" ```

## Тесты программы

``` pytest test_git_dep.py ```

![image](https://github.com/user-attachments/assets/277812d0-93a0-40e0-b3fb-a31dabf5d236)
