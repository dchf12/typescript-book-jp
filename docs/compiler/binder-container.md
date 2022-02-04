### Container

AST nodeはContainerになる可能性もあります。これは、Nodeとそれに関連するSymbolが持つ`SymbolTables`の種類が決定します。Containerは抽象的な概念です(つまり、関連するデータ構造はありません)。この概念は、いくつかのものによって駆動されます。1つは`ContainerFlags`のenumです。関数`getContainerFlags`(`binder.ts`にあります)がこのフラグを駆動します。それを下記に示します:

```ts
function getContainerFlags(node: Node): ContainerFlags {
    switch (node.kind) {
        case SyntaxKind.ClassExpression:
        case SyntaxKind.ClassDeclaration:
        case SyntaxKind.InterfaceDeclaration:
        case SyntaxKind.EnumDeclaration:
        case SyntaxKind.TypeLiteral:
        case SyntaxKind.ObjectLiteralExpression:
            return ContainerFlags.IsContainer;

        case SyntaxKind.CallSignature:
        case SyntaxKind.ConstructSignature:
        case SyntaxKind.IndexSignature:
        case SyntaxKind.MethodDeclaration:
        case SyntaxKind.MethodSignature:
        case SyntaxKind.FunctionDeclaration:
        case SyntaxKind.Constructor:
        case SyntaxKind.GetAccessor:
        case SyntaxKind.SetAccessor:
        case SyntaxKind.FunctionType:
        case SyntaxKind.ConstructorType:
        case SyntaxKind.FunctionExpression:
        case SyntaxKind.ArrowFunction:
        case SyntaxKind.ModuleDeclaration:
        case SyntaxKind.SourceFile:
        case SyntaxKind.TypeAliasDeclaration:
            return ContainerFlags.IsContainerWithLocals;

        case SyntaxKind.CatchClause:
        case SyntaxKind.ForStatement:
        case SyntaxKind.ForInStatement:
        case SyntaxKind.ForOfStatement:
        case SyntaxKind.CaseBlock:
            return ContainerFlags.IsBlockScopedContainer;

        case SyntaxKind.Block:
            // do not treat blocks directly inside a function as a block-scoped-container.
            // Locals that reside in this block should go to the function locals. Otherwise 'x'
            // would not appear to be a redeclaration of a block scoped local in the following
            // example:
            //
            //      function foo() {
            //          var x;
            //          let x;
            //      }
            //
            // If we placed 'var x' into the function locals and 'let x' into the locals of
            // the block, then there would be no collision.
            //
            // By not creating a new block-scoped-container here, we ensure that both 'var x'
            // and 'let x' go into the Function-container's locals, and we do get a collision
            // conflict.
            return isFunctionLike(node.parent) ? ContainerFlags.None : ContainerFlags.IsBlockScopedContainer;
    }

    return ContainerFlags.None;
}
```

これはBinderの`bindChildren`関数から*のみ*呼び出され、nodeを`container`として、もしくは、`blockScopedContainer`として設定します。これは`getContainerFlags`関数の評価に依存します。`bindChildren`を以下に示します：

```ts
// All container nodes are kept on a linked list in declaration order. This list is used by
// the getLocalNameOfContainer function in the type checker to validate that the local name
// used for a container is unique.
function bindChildren(node: Node) {
    // Before we recurse into a node's children, we first save the existing parent, container
    // and block-container.  Then after we pop out of processing the children, we restore
    // these saved values.
    let saveParent = parent;
    let saveContainer = container;
    let savedBlockScopeContainer = blockScopeContainer;

    // This node will now be set as the parent of all of its children as we recurse into them.
    parent = node;

    // Depending on what kind of node this is, we may have to adjust the current container
    // and block-container.   If the current node is a container, then it is automatically
    // considered the current block-container as well.  Also, for containers that we know
    // may contain locals, we proactively initialize the .locals field. We do this because
    // it's highly likely that the .locals will be needed to place some child in (for example,
    // a parameter, or variable declaration).
    //
    // However, we do not proactively create the .locals for block-containers because it's
    // totally normal and common for block-containers to never actually have a block-scoped
    // variable in them.  We don't want to end up allocating an object for every 'block' we
    // run into when most of them won't be necessary.
    //
    // Finally, if this is a block-container, then we clear out any existing .locals object
    // it may contain within it.  This happens in incremental scenarios.  Because we can be
    // reusing a node from a previous compilation, that node may have had 'locals' created
    // for it.  We must clear this so we don't accidentally move any stale data forward from
    // a previous compilation.
    let containerFlags = getContainerFlags(node);
    if (containerFlags & ContainerFlags.IsContainer) {
        container = blockScopeContainer = node;

        if (containerFlags & ContainerFlags.HasLocals) {
            container.locals = {};
        }

        addToContainerChain(container);
    }

    else if (containerFlags & ContainerFlags.IsBlockScopedContainer) {
        blockScopeContainer = node;
        blockScopeContainer.locals = undefined;
    }

    forEachChild(node, bind);

    container = saveContainer;
    parent = saveParent;
    blockScopeContainer = savedBlockScopeContainer;
}
```

Binder関数のセクションから思い出されるように`bind`関数から`bindChildren`が呼び出されます。だから我々は再帰的なbindingの設定が行えます。つまり、`bind`は、`bindChildren`を呼び出し、それはそれぞれの子に対して`bind`を呼び出します。
