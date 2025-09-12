package.json

{
  "name": "mitra",
  "displayName": "Mitra Assistant",
  "description": "An AI-powered metadata platform helping tool.",
  "version": "0.0.1",
  "engines": {
    "vscode": "^1.80.0"
  },
  "categories": ["Other"],
  "activationEvents": [
    "*"
  ],
  "main": "./dist/extension.js",
  "contributes": {
    "configuration": {
      "title": "Mitra Assistant",
      "properties": {
        "mitra.prompts": {
          "type": "array",
          "description": "List of custom prompts for Mitra.",
          "items": {
            "type": "object",
            "properties": {
              "name": {
                "type": "string",
                "description": "Name of the prompt (used to call it)."
              },
              "promptText": {
                "type": "string",
                "description": "The actual AI/system prompt instruction."
              },
              "inputFolder": {
                "type": "string",
                "description": "Path to input folder."
              },
              "outputFolder": {
                "type": "string",
                "description": "Path to output folder."
              }
            },
            "required": ["name", "promptText", "inputFolder", "outputFolder"]
          }
        }
      }
    }
  },
  "scripts": {
    "vscode:prepublish": "npm run compile",
    "compile": "esbuild ./src/extension.ts --bundle --outdir=dist --platform=node --external:vscode --format=cjs",
    "watch": "esbuild ./src/extension.ts --bundle --outdir=dist --platform=node --external:vscode --format=cjs --watch",
    "package": "vsce package"
  },
  "devDependencies": {
    "@types/node": "^18.0.0",
    "@types/vscode": "^1.80.0",
    "esbuild": "^0.19.0",
    "typescript": "^5.0.0"
  }
}


************** extension ts *****

import * as vscode from 'vscode';
import * as fs from 'fs';
import * as path from 'path';

export function activate(context: vscode.ExtensionContext) {
    const config = vscode.workspace.getConfiguration("mitra");
    const prompts = config.get<any[]>("prompts") || [];

    if (prompts.length === 0) {
        vscode.window.showInformationMessage("Mitra: No prompts configured. Please add prompts in settings.");
        return;
    }

    prompts.forEach(prompt => {
        const commandName = `mitra.${prompt.name}`;
        let disposable = vscode.commands.registerCommand(commandName, async () => {
            vscode.window.showInformationMessage(`Mitra running prompt: ${prompt.name}`);

            try {
                if (!fs.existsSync(prompt.inputFolder)) {
                    vscode.window.showErrorMessage(`Input folder does not exist: ${prompt.inputFolder}`);
                    return;
                }
                if (!fs.existsSync(prompt.outputFolder)) {
                    fs.mkdirSync(prompt.outputFolder, { recursive: true });
                }

                const inputFiles = fs.readdirSync(prompt.inputFolder);

                for (const file of inputFiles) {
                    const filePath = path.join(prompt.inputFolder, file);
                    const content = fs.readFileSync(filePath, 'utf-8');

                    // Stub: Here you’d call AI model with prompt.promptText + content
                    const processed = `🔹 Mitra Prompt: ${prompt.promptText}\n\n---\n\n${content}`;

                    const outputPath = path.join(prompt.outputFolder, file);
                    fs.writeFileSync(outputPath, processed, 'utf-8');
                }

                vscode.window.showInformationMessage(`Prompt "${prompt.name}" completed! Results saved in ${prompt.outputFolder}`);
            } catch (err: any) {
                vscode.window.showErrorMessage(`Mitra Error: ${err.message}`);
            }
        });

        context.subscriptions.push(disposable);
    });
}

export function deactivate() {}


****** settings json ****

"mitra.prompts": [
  {
    "name": "summarizeCode",
    "promptText": "Summarize all functions in plain English.",
    "inputFolder": "C:/Users/kumar/projects/input",
    "outputFolder": "C:/Users/kumar/projects/output"
  },
  {
    "name": "checkSQL",
    "promptText": "Validate all SQL queries for best practices.",
    "inputFolder": "C:/Users/kumar/sqlfiles",
    "outputFolder": "C:/Users/kumar/sqlout"
  }
]