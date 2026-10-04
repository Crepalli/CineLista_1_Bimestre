# Recursos avançados mantidos no projeto

Alguns recursos avançados são necessários para manter a organização dos componentes e funcionalidades importantes do CineLista.

## 1. `@Input`

```typescript
@Input() filmes: Filme[] = [];
```

### Por que foi mantido

O array principal de títulos pertence ao componente raiz `App`, mas os itens são exibidos pelo componente `CatalogoFilmes`.

O `@Input` permite que o componente filho receba informações do componente pai.

```html
<app-catalogo-filmes [filmes]="filmes" />
```

### Como funciona

O componente raiz envia o array por meio do property binding `[filmes]`. O catálogo recebe esse valor na propriedade marcada com `@Input`.

```text
App → envia o array → CatalogoFilmes
```

Sem o `@Input`, seria necessário duplicar o array, colocar tudo em um único componente ou utilizar um serviço compartilhado.

## 2. `@Output` e `EventEmitter`

```typescript
@Output() filmeRemovido = new EventEmitter<number>();
```

### Por que foram mantidos

O catálogo pode solicitar a remoção ou alteração de um título, mas o array pertence ao componente raiz. Por isso, o componente filho precisa avisar o componente pai sobre a ação.

### Como funciona

O catálogo emite o ID:

```typescript
this.filmeRemovido.emit(id);
```

O componente raiz recebe esse valor pelo `$event`:

```html
(filmeRemovido)="removerFilme($event)"
```

O mesmo processo é utilizado no formulário:

```html
<app-formulario-filme
  (filmeAdicionado)="adicionarFilme($event)"
/>
```

O fluxo é:

```text
Componente filho → emite o evento → componente pai executa a alteração
```

O `EventEmitter<number>` informa que o evento enviará um número. No caso da remoção, esse número é o ID do título.

Já o formulário utiliza:

```typescript
EventEmitter<NovoFilme>
```

Nesse caso, o evento envia os dados completos do novo título.

## 3. `$event`

```html
(filmeRemovido)="removerFilme($event)"
```

### Por que foi mantido

O `$event` é necessário para receber o valor enviado pelo componente filho ou as informações de um evento HTML.

### Como funciona

Em um evento personalizado, o `$event` contém o valor enviado pelo `emit()`.

```typescript
this.filmeRemovido.emit(id);
```

Nesse exemplo, `$event` representa o ID.

Em um evento HTML:

```html
<input type="file" (change)="selecionarImagem($event)" />
```

O `$event` contém informações sobre o campo que disparou o evento.

## 4. `FileReader`

```typescript
const leitor = new FileReader();

leitor.onload = () => {
  this.imagemUrl = String(leitor.result);
};

leitor.readAsDataURL(arquivo);
```

### Por que foi mantido

O `FileReader` é necessário para permitir que o usuário selecione uma imagem armazenada no computador.

O navegador não permite utilizar diretamente o caminho local do arquivo por questões de segurança.

### Como funciona

1. O usuário seleciona uma imagem.
2. O evento `(change)` chama o método `selecionarImagem()`.
3. O método recebe o arquivo selecionado.
4. O `FileReader` lê o conteúdo da imagem.
5. `readAsDataURL()` transforma o arquivo em uma URL de dados.
6. Quando a leitura termina, o evento `onload` é executado.
7. O resultado é armazenado em `imagemUrl`.
8. A imagem é apresentada com `[src]="imagemUrl"`.

Se o `FileReader` fosse removido, o formulário poderia aceitar somente uma URL de imagem, perdendo a opção de selecionar um arquivo do computador.

## 5. Conversão do elemento HTML

```typescript
const campo = evento.target as HTMLInputElement;
```

Também é utilizada nas imagens:

```typescript
const imagem = evento.target as HTMLImageElement;
```

### Por que foi mantida

Para o TypeScript, `evento.target` é apenas um elemento genérico. Ele não sabe automaticamente se o elemento é um input ou uma imagem.

### Como funciona

O comando:

```typescript
as HTMLInputElement
```

informa ao TypeScript que o elemento é um campo `<input>`. Assim, o código pode acessar a propriedade:

```typescript
campo.files
```

Já:

```typescript
as HTMLImageElement
```

informa que o elemento é uma imagem, permitindo acessar:

```typescript
imagem.src
```

Essa conversão não modifica o elemento. Ela apenas informa seu tipo ao TypeScript.

## 6. Tratamento de erro da imagem

```html
<img
  [src]="filme.imagemUrl"
  (error)="usarCapaPadrao($event)"
/>
```

```typescript
usarCapaPadrao(evento: Event): void {
  const imagem = evento.target as HTMLImageElement;
  imagem.src = '/imagens/capa-padrao.svg';
}
```

### Por que foi mantido

Uma imagem externa pode deixar de existir, ficar indisponível ou possuir uma URL incorreta. Sem tratamento, o navegador mostraria o símbolo de imagem quebrada.

### Como funciona

Quando a imagem não carrega, o navegador dispara o evento `(error)`. O método recebe a imagem que falhou e troca seu endereço pela capa padrão do projeto.

```text
Imagem falhou → evento error → troca pela capa padrão
```

## 7. `stopPropagation()`

```html
<div class="modal-fundo" (click)="fecharDetalhes()">
  <article class="modal" (click)="$event.stopPropagation()">
```

### Por que foi mantido

O fundo do modal fecha a janela quando é clicado. Porém, sem o `stopPropagation()`, clicar no conteúdo interno também acionaria o evento do fundo.

### Como funciona

Os eventos normalmente passam do elemento interno para o elemento externo. Esse comportamento é chamado de propagação.

O comando:

```typescript
$event.stopPropagation()
```

interrompe essa propagação.

O resultado é:

- clique fora do conteúdo: fecha o modal;
- clique dentro do conteúdo: mantém o modal aberto;
- clique no botão `X`: fecha o modal.

Se esse comando fosse removido, seria necessário retirar o fechamento ao clicar no fundo e permitir o fechamento somente pelo botão `X`.


informa ao Angular para utilizar esse ID como identificador.

Quando um título é adicionado, removido ou alterado, o Angular consegue descobrir exatamente qual item mudou, sem precisar recriar toda a lista.

Para listas simples de textos, o próprio valor pode ser utilizado:


## 8. Conclusão

Os recursos avançados foram mantidos, pois possuem funções específicas que seriam difíceis de substituir sem:

- perder funcionalidades;
- juntar todos os códigos em um único componente;
- duplicar informações;
- ou adicionar soluções ainda mais avançadas.

Eles foram utilizados principalmente para:

- comunicação entre componentes;
- envio de informações pelos eventos;
- seleção de imagem local;
- tratamento de imagens inválidas;
- funcionamento correto do modal;
- renderização eficiente das listas.
