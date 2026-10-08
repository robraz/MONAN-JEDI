# Procedimentos de Compilação, Execução e CTest do MONAN-JEDI na JACI

O **MONAN-JEDI** pode ser compilado e testado na plataforma JACI utilizando uma adaptação do código-fonte para integrar o modelo MONAN através de um repositório forkado. Este documento descreve o procedimento completo de obtenção do código, compilação, execução dos testes (CTest) e correção dos problemas encontrados durante a validação.

## Obtenção do código-fonte

Diretório de trabalho:

```text
/lustre/projetos/monan_das/rodrigo.braz/src/spack-stack
```

Clonar o repositório:

```bash
git clone https://github.com/robraz/MONAN-JEDI.git
cd MONAN-JEDI
```

## Ajuste da configuração do modelo MPAS

Antes da compilação, é necessário alterar a referência do modelo MPAS definida em `CMakeLists.txt`.

Substituir:

```cmake
ecbuild_bundle(
  PROJECT mpas
  GIT "https://github.com/MPAS-Dev/MPAS-Model.git"
  TAG 0e5a47a0e1bcccd6e3d99909b76e740a643c4db6
)
```

Por:

```cmake
ecbuild_bundle(
  PROJECT mpas
  GIT "https://github.com/robraz/MONAN-Model.git"
  TAG v1.0.0-monan
)
```

### Observação

O repositório:

```text
https://github.com/robraz/MONAN-Model.git
```

é um fork utilizado especificamente para a compilação do MONAN-JEDI.

Repositório original:

```text
https://github.com/GAD-DIMNT-CPTEC/MONAN-JEDI
```

As modificações necessárias para compatibilidade e compilação do sistema MONAN-JEDI foram implementadas nesse fork.

## Compilação e instalação

Executar:

```bash
bash scripts/monan-jedi.sh all --config config/jaci.yaml
```

Resultado obtido:

```text
100% OK
```

## Execução do CTest

Executar:

```bash
bash scripts/monan-jedi.sh test-pbs --config config/jaci.yaml
```

Resultado inicial:

```text
97% tests passed, 60 tests failed out of 2294
```

### Primeiro erro identificado

```text
Erro 2235/2294
```

O teste falha porque a variável:

```text
isctyp
```

não é encontrada no arquivo:

```text
x1.2562.invariant.nc
```

### Resolução

Adicionar a variável `isctyp` no arquivo:

```text
/lustre/projetos/monan_das/rodrigo.braz/src/spack-stack/MONAN-JEDI/mpas-jedi-data/testinput_tier_1/480km/bg/x1.2562.invariant.nc
```

Executar novamente:

```bash
bash scripts/monan-jedi.sh test-pbs --config config/jaci.yaml
```

Resultado:

```text
99% tests passed, 26 tests failed out of 2294
```

## Segunda ocorrência do problema

### Primeiro erro remanescente

```text
Erro 2245/2294
```

Novamente, a variável:

```text
isctyp
```

não é encontrada no arquivo:

```text
x1.4002.invariant.nc
```

### Resolução

Adicionar a variável `isctyp` no arquivo:

```text
/lustre/projetos/monan_das/rodrigo.braz/src/spack-stack/MONAN-JEDI/mpas-jedi-data/testinput_tier_1/384km/bg/x1.4002.invariant.nc
```

Executar novamente os testes:

```bash
bash scripts/monan-jedi.sh test-pbs --config config/jaci.yaml
```

Resultado:

```text
99% tests passed, 2 tests failed out of 2294
```

## Falhas remanescentes

Os únicos testes restantes com falha foram:

```text
2256 - mpasjedi_parameters_bumploc_regional
2272 - mpasjedi_3denvar_multi_resolution_regional
```

Ambos falharam pelo mesmo motivo: ausência da variável `isctyp` em arquivos de dados invariantes.

### Resolução

Adicionar a variável `isctyp` nos arquivos:

```text
/lustre/projetos/monan_das/rodrigo.braz/src/spack-stack/MONAN-JEDI/mpas-jedi-data/testinput_tier_1/384km/bg/conus384km.invariant.nc
```

e

```text
/lustre/projetos/monan_das/rodrigo.braz/src/spack-stack/MONAN-JEDI/mpas-jedi-data/testinput_tier_1/480km/bg/conus480km.invariant.nc
```

## Validação final

Executar novamente:

```bash
cd /lustre/projetos/monan_das/rodrigo.braz/src/spack-stack/MONAN-JEDI

bash scripts/monan-jedi.sh test-pbs --config config/jaci.yaml
```

Resultado final:

```text
100% tests passed, 0 tests failed out of 2294
```

## Resumo das correções aplicadas

Todos os erros encontrados durante a execução do CTest estavam relacionados à ausência da variável:

```text
isctyp
```

nos arquivos NetCDF invariantes utilizados pelos testes do MPAS-JEDI. Os arquivos corrigidos foram:

```text
mpas-jedi-data/testinput_tier_1/480km/bg/x1.2562.invariant.nc

mpas-jedi-data/testinput_tier_1/384km/bg/x1.4002.invariant.nc

mpas-jedi-data/testinput_tier_1/384km/bg/conus384km.invariant.nc

mpas-jedi-data/testinput_tier_1/480km/bg/conus480km.invariant.nc
```

Após a inclusão da variável nesses arquivos, a suíte completa de **2294 testes** foi executada com sucesso, resultando em:

```text
100% tests passed, 0 tests failed out of 2294
```
