# CMake Tutorial

Este tutorial cobre o seguinte tópicos:

1. Construção de projetos usando makefiles.
2. Construção de projetos usando `cmake(1)`.
3. Construção de projetos usando makefiles `cmake(1)` com bibliotecas de terceiros.

Neste tutorial, usaremos a seguinte estrutura de projeto:

```
cmake-tutorial/
├── CMakeLists.txt
├── README.md
├── src
│   ├── main.cc
│   ├── math.cc
│   └── math.h
└── test
    └── math_test.cc
```

Diretórios:

- `src`  : diretório para código-fonte.
- `test` : diretório para teste.

`src/main.cc` é o programa principal e `src/math.{cc,h}` é uma biblioteca interna usada em `src/main.cc`.

O primeiro passo deste tutorial é contruir o projeto usando um simples makefile. 
Em seguida, definiremos um script CMakeLists.txt que gerará complexo Makefiles para nós
atraves do comando `cmake`.


## Instalando CMake

A instalação do `cmake` geralmente é um processo simples.  

No Ubuntu:

```
sudo apt-get install cmake
```

No macOS:

```
brew install cmake
```

No windows:
Recomenda utilizar o subsistema linux no windows e proceder com a instalação da mesma forma que no linux.

Veja o seguinte link:
 - https://docs.microsoft.com/pt-br/windows/wsl/setup/environment

Verifique se a instalação está correta: 

```
% cmake --version
cmake version 3.10.2

CMake suite maintained and supported by Kitware (kitware.com/cmake).
```

## Compilando & Link-editando

Podemos compilar esse projeto manualmente usando o comando

```
g++ src/main.cc src/math.cc -o cmake-tutorial
```

Ou podemos compilar e link-editar em etapas separadas

```
g++ -c src/math.cc -o math.o 
g++ src/main.cc math.o -o cmake-tutorial
```

## Usando Makefile

Pode-se automatizar o processo de compilação e link-edição acima usando `Makefile`.
Primeiro temos que criar o arquivo `Makefile` no diretório raiz com o seguinte conteúdo:

```makefile
# Add definition to generate math.o object file
math.o: src/math.cc src/math.h
    c++ -c src/math.cc -o math.o

# Add definition to generate cmake-tutorial binary
cmake-tutorial: math.o
    c++ src/main.cc math.o -o cmake-tutorial
```

Agora podemos executar:

```
make cmake-tutorial
```

para contruir o executável `cmake-tutorial`. Se não houver nenhuma mudança nos arquivos `src/{main,math}.cc` e `src/math.h`,
o comando a seguir não produzira nehuma ação:

```
% make cmake-tutorial
make: Nothing to be done for `cmake-tutorial'.
```

Isto é extremamente util quando trabalhamos com projetos muito grandes um fez que apenas arquivos com alterações no código serão recompilados.

## Usando CMake

Agora podemos usar `cmake` para fazer todo esse processo para nós.

Vamos criar um arquivo `CMakeLists.txt` com o seguinte conteúdo:

```cmake
cmake_minimum_required (VERSION 3.10)

# Define o projeto
project(cmake-tutorial)

# Define a biblioteca local com nome math
add_library(math src/math.cc)

# Define o executável 
add_executable(cmake-tutorial src/main.cc)

# Indique que a biblioteca math deve ser link-editada com o executável
target_link_libraries(cmake-tutorial math)
```

Podemos gerar um `Makefile` baseado na definição acima usando o seguinte comando: 

```
cmake .
```

Ou crie u diretório de `build` para armazenar os arquivos gerados pelo cmake

```
mkdir build
cd build/
cmake ..
```

Agora podemos executar o comando `make cmake-tutorial` para gerar o executável.

```
% make cmake-tutorial
Scanning dependencies of target math
[ 25%] Building CXX object CMakeFiles/math.dir/src/math.cc.o
[ 50%] Linking CXX static library libmath.a
[ 50%] Built target math
Scanning dependencies of target cmake-tutorial
[ 75%] Building CXX object CMakeFiles/cmake-tutorial.dir/src/main.cc.o
[100%] Linking CXX executable cmake-tutorial
[100%] Built target cmake-tutorial
```

Ou podemos usar o CMake diretamente via:

```
cmake --build . --target cmake-tutorial
```

## Usando o CMake com biblioteca de Terceiros


Suponha que queremos escrever um teste unitário para `math::add(a, b)`.
Usaremos a biblioteca [googletest](https://github.com/google/googletest) para criar e executar o teste.

Adicione a seguinte definição a `CMakeLists.txt`:

```cmake
# Third-party library
include(ExternalProject)
ExternalProject_Add(googletest
    PREFIX "${CMAKE_BINARY_DIR}/lib"
    GIT_REPOSITORY "https://github.com/google/googletest.git"
    GIT_TAG "master"
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/lib/installed
)

# Prevent build on all targets build
set_target_properties(googletest PROPERTIES EXCLUDE_FROM_ALL TRUE)

# Define ${CMAKE_INSTALL_...} variables
include(GNUInstallDirs)

# Specify where third-party libraries are located
link_directories(${CMAKE_BINARY_DIR}/lib/installed/${CMAKE_INSTALL_LIBDIR})
include_directories(${CMAKE_BINARY_DIR}/lib/installed/${CMAKE_INSTALL_INCLUDEDIR})

# This is required for googletest
find_package(Threads REQUIRED)

# Test
add_executable(math_test test/math_test.cc)
target_link_libraries(math_test math gtest Threads::Threads)
# Make sure third-party is built before executable
add_dependencies(math_test googletest)
set_target_properties(math_test PROPERTIES EXCLUDE_FROM_ALL TRUE)
```
Gera o makefile novamente:

```
cd build/
cmake ..
```

Gera o teste unitário:

```
cmake --build . --target math_test
```

Executa o teste:

```
% ./math_test 
[==========] Running 6 tests from 3 test cases.
[----------] Global test environment set-up.
[----------] 2 tests from MathAddTest
[ RUN      ] MathAddTest.PositiveNum
[       OK ] MathAddTest.PositiveNum (0 ms)
[ RUN      ] MathAddTest.ZeroB
[       OK ] MathAddTest.ZeroB (0 ms)
[----------] 2 tests from MathAddTest (0 ms total)

[----------] 2 tests from MathSubTest
[ RUN      ] MathSubTest.PositiveNum
[       OK ] MathSubTest.PositiveNum (0 ms)
[ RUN      ] MathSubTest.ZeroB
[       OK ] MathSubTest.ZeroB (0 ms)
[----------] 2 tests from MathSubTest (0 ms total)

[----------] 2 tests from MathMulTest
[ RUN      ] MathMulTest.PositiveNum
[       OK ] MathMulTest.PositiveNum (0 ms)
[ RUN      ] MathMulTest.ZeroB
[       OK ] MathMulTest.ZeroB (0 ms)
[----------] 2 tests from MathMulTest (0 ms total)

[----------] Global test environment tear-down
[==========] 6 tests from 3 test cases ran. (0 ms total)
[  PASSED  ] 6 tests.
```

Done.


### IDE Support

Se você usa`CLion`, o google test será automaticamente detectado.

![CLion](https://s9.postimg.org/ugqkdw6nh/Screen_Shot_2018-02-16_at_21.03.10.png)


