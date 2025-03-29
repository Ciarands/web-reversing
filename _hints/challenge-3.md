# Steps to defeat:

<details>
  <summary>Step 1</summary>
  
  Iterate all functions and index the `length` and the `.toString()`.`length` of each.
</details>

<details>
  <summary>Step 2</summary>
  
  Replace `func['length']` with the indexed length and `[func+[]][+[]]['length']` with the toStringed length
</details>

<details>
  <summary>Step 3</summary>
  
  Arbitrary modification is now possible, you can now inline all operation functions regardless of parameter count e.g 

  function mult(a, b, c, e) { return a * b } -> a * b
</details>

<details>
  <summary>Step 4</summary>
  
  Isolate the statements that direct to each branch.

  Either modify the statement or extract the specific charcodes which contain the flag.
</details>

<details>
  <summary>Partial Deobfuscator Example</summary>
  
  ```js
  const parser = require('@babel/parser');
  const traverse = require('@babel/traverse').default;
  const t = require('@babel/types');
  const generate = require('@babel/generator').default;

  function getFuncIdentifier(node) {
    const innerMember = node.object;
    const arrayExpr = innerMember.object;
    const binExpr = arrayExpr.elements[0];
    return binExpr.left;
  }

  function isToStringLengthAccess(node) {
    if (
      !t.isMemberExpression(node) ||
      !node.computed ||
      !t.isStringLiteral(node.property, {
        value: 'length'
      })
    )
      return false;

    const innerMember = node.object;
    if (!t.isMemberExpression(innerMember)) return false;

    const arrayExpr = innerMember.object;
    if (!t.isArrayExpression(arrayExpr) || arrayExpr.elements.length !== 1)
      return false;

    const binExpr = arrayExpr.elements[0];
    if (!t.isBinaryExpression(binExpr) || binExpr.operator !== '+') return false;

    if (!t.isIdentifier(binExpr.left) || !t.isArrayExpression(binExpr.right) || binExpr.right.elements.length !== 0)
      return false;

    const unaryExpr = innerMember.property;
    if (
      !t.isUnaryExpression(unaryExpr) ||
      unaryExpr.operator !== '+' ||
      !unaryExpr.prefix
    )
      return false;

    const unaryArg = unaryExpr.argument;
    if (!t.isArrayExpression(unaryArg) || unaryArg.elements.length !== 0)
      return false;

    return true;
  }


  function transform(code) {
    const ast = parser.parse(code, {
      sourceType: 'script',
      ranges: true,
    });

    const functionData = new Map();
    const inlineableFunctions = new Map();

    traverse(ast, {
      FunctionDeclaration(path) {
        const {
          node
        } = path;
        const name = node.id?.name;
        if (!name) return;

        const paramsLength = node.params.length;
        const funcCode = code.slice(node.start, node.end);
        functionData.set(name, {
          paramsLength,
          toStringLength: funcCode.length,
        });

        let returnExpr;
        const body = node.body;
        if (body.type === 'BlockStatement' && body.body.length === 1) {
          const stmt = body.body[0];
          if (stmt.type === 'ReturnStatement' && stmt.argument) {
            returnExpr = stmt.argument;
          }
        }

        if (!returnExpr) return;

        const paramNames = node.params.map(p => p.name);
        let inlineableData;

        if (returnExpr.type === 'BinaryExpression') {
          const left = returnExpr.left;
          const right = returnExpr.right;
          if (left.type === 'Identifier' && right.type === 'Identifier' &&
            paramNames.includes(left.name) && paramNames.includes(right.name)) {
            inlineableData = {
              type: 'binary',
              operator: returnExpr.operator,
              leftParamIndex: paramNames.indexOf(left.name),
              rightParamIndex: paramNames.indexOf(right.name),
            };
          }
        } else if (returnExpr.type === 'CallExpression') {
          const callee = returnExpr.callee;
          if (callee.type === 'Identifier') {
            const argsValid = returnExpr.arguments.every(arg => {
              return arg.type === 'Identifier' && paramNames.includes(arg.name);
            });
            if (argsValid) {
              inlineableData = {
                type: 'call',
                callee: callee.name,
                args: returnExpr.arguments.map(arg => paramNames.indexOf(arg.name)),
              };
            }
          }
        }

        if (inlineableData) {
          inlineableFunctions.set(name, {
            data: inlineableData,
            path: path
          });
        }
      },
      FunctionExpression(path) {
        const {
          node
        } = path;
        let name;
        let pathToRemove = path;
        const parent = path.parent;

        if (t.isVariableDeclarator(parent) && t.isIdentifier(parent.id)) {
          name = parent.id.name;
          pathToRemove = path.parentPath;
        } else if (t.isAssignmentExpression(parent) && t.isIdentifier(parent.left)) {
          name = parent.left.name;
          pathToRemove = path.parentPath;
        } else {
          return;
        }

        const paramsLength = node.params.length;
        const funcCode = code.slice(node.start, node.end);
        functionData.set(name, {
          paramsLength,
          toStringLength: funcCode.length,
        });

        let returnExpr;
        const body = node.body;
        if (body.type === 'BlockStatement' && body.body.length === 1) {
          const stmt = body.body[0];
          if (stmt.type === 'ReturnStatement' && stmt.argument) {
            returnExpr = stmt.argument;
          }
        } else if (node.expression) {
          returnExpr = body;
        }

        if (!returnExpr) return;

        const paramNames = node.params.map(p => p.name);
        let inlineableData;

        if (returnExpr.type === 'BinaryExpression') {
          const left = returnExpr.left;
          const right = returnExpr.right;
          if (left.type === 'Identifier' && right.type === 'Identifier' &&
            paramNames.includes(left.name) && paramNames.includes(right.name)) {
            inlineableData = {
              type: 'binary',
              operator: returnExpr.operator,
              leftParamIndex: paramNames.indexOf(left.name),
              rightParamIndex: paramNames.indexOf(right.name),
            };
          }
        } else if (returnExpr.type === 'CallExpression') {
          const callee = returnExpr.callee;
          if (callee.type === 'Identifier') {
            const argsValid = returnExpr.arguments.every(arg => {
              return arg.type === 'Identifier' && paramNames.includes(arg.name);
            });
            if (argsValid) {
              inlineableData = {
                type: 'call',
                callee: callee.name,
                args: returnExpr.arguments.map(arg => paramNames.indexOf(arg.name)),
              };
            }
          }
        }

        if (inlineableData) {
          inlineableFunctions.set(name, {
            data: inlineableData,
            path: pathToRemove
          });
        }
      },
      ArrowFunctionExpression(path) {
        const {
          node
        } = path;
        let name;
        let pathToRemove = path;
        const parent = path.parent;

        if (t.isVariableDeclarator(parent) && t.isIdentifier(parent.id)) {
          name = parent.id.name;
          pathToRemove = path.parentPath;
        } else if (t.isAssignmentExpression(parent) && t.isIdentifier(parent.left)) {
          name = parent.left.name;
          pathToRemove = path.parentPath;
        } else {
          return;
        }

        const paramsLength = node.params.length;
        const funcCode = code.slice(node.start, node.end);
        functionData.set(name, {
          paramsLength,
          toStringLength: funcCode.length,
        });

        let returnExpr = node.body;
        if (node.body.type === 'BlockStatement') {
          if (node.body.body.length === 1) {
            const stmt = node.body.body[0];
            if (stmt.type === 'ReturnStatement' && stmt.argument) {
              returnExpr = stmt.argument;
            }
          } else {
            return;
          }
        }

        const paramNames = node.params.map(p => p.name);
        let inlineableData;

        if (returnExpr.type === 'BinaryExpression') {
          const left = returnExpr.left;
          const right = returnExpr.right;
          if (left.type === 'Identifier' && right.type === 'Identifier' &&
            paramNames.includes(left.name) && paramNames.includes(right.name)) {
            inlineableData = {
              type: 'binary',
              operator: returnExpr.operator,
              leftParamIndex: paramNames.indexOf(left.name),
              rightParamIndex: paramNames.indexOf(right.name),
            };
          }
        } else if (returnExpr.type === 'CallExpression') {
          const callee = returnExpr.callee;
          if (callee.type === 'Identifier') {
            const argsValid = returnExpr.arguments.every(arg => {
              return arg.type === 'Identifier' && paramNames.includes(arg.name);
            });
            if (argsValid) {
              inlineableData = {
                type: 'call',
                callee: callee.name,
                args: returnExpr.arguments.map(arg => paramNames.indexOf(arg.name)),
              };
            }
          }
        }

        if (inlineableData) {
          inlineableFunctions.set(name, {
            data: inlineableData,
            path: pathToRemove
          });
        }
      },
    });

    traverse(ast, {
      MemberExpression(path) {
        const {
          node
        } = path;

        if (
          node.computed &&
          t.isStringLiteral(node.property, {
            value: 'length'
          }) &&
          t.isIdentifier(node.object)
        ) {
          const funcName = node.object.name;
          const data = functionData.get(funcName);
          if (data) {
            path.replaceWith(t.numericLiteral(data.paramsLength));
            path.skip();
          }
          return;
        }

        console.log(isToStringLengthAccess(node));

        if (isToStringLengthAccess(node)) {
          const funcIdentifier = getFuncIdentifier(node);
          if (funcIdentifier && t.isIdentifier(funcIdentifier)) {
            const funcName = funcIdentifier.name;
            const data = functionData.get(funcName);
            if (data) {
              path.replaceWith(t.numericLiteral(data.toStringLength));
              path.skip();
            }
          }
        }
      },
      CallExpression(path) {
        const {
          node
        } = path;
        if (t.isIdentifier(node.callee)) {
          const funcName = node.callee.name;
          const entry = inlineableFunctions.get(funcName);
          if (entry) {
            const {
              data
            } = entry;
            if (data.type === 'binary') {
              const leftArg = node.arguments[data.leftParamIndex];
              const rightArg = node.arguments[data.rightParamIndex];
              if (leftArg && rightArg) {
                const binaryExpr = t.binaryExpression(data.operator, leftArg, rightArg);
                path.replaceWith(binaryExpr);
              }
            } else if (data.type === 'call') {
              const newCallee = t.identifier(data.callee);
              const newArgs = data.args.map(argIndex => node.arguments[argIndex]).filter(arg => arg);
              const callExpr = t.callExpression(newCallee, newArgs);
              path.replaceWith(callExpr);
            }
          }
        }
      },
    });

    inlineableFunctions.forEach(({
      path
    }) => {
      path.remove();
    });

    return generate(ast).code;
  }
  ```
</details>