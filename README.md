# CPUの動作

import React, { useState } from 'react';
import { Play, RotateCcw, StepForward, BookOpen, Plus, Trash2 } from 'lucide-react';

const CPUSimulator = () => {
  const [registers, setRegisters] = useState({ A: 0, B: 0 });
  const [memory, setMemory] = useState(Array(10).fill(0));
  const [programCounter, setProgramCounter] = useState(0);
  const [instructionRegister, setInstructionRegister] = useState('');
  const [isRunning, setIsRunning] = useState(false);
  const [log, setLog] = useState([]);
  const [program, setProgram] = useState([
    { cmd: 'READ', arg1: 'A', arg2: '0' },
    { cmd: 'READ', arg1: 'B', arg2: '1' },
    { cmd: 'ADD', arg1: 'A', arg2: 'B' },
    { cmd: 'WRITE', arg1: 'A', arg2: '2' },
    { cmd: 'STOP', arg1: '', arg2: '' }
  ]);

  const executeInstruction = (instruction) => {
    const { cmd, arg1, arg2 } = instruction;
    let message = '';
    
    switch(cmd) {
      case 'READ':
        const addr = parseInt(arg2);
        const value = memory[addr];
        setRegisters(prev => ({ ...prev, [arg1]: value }));
        message = `メモリ[${addr}]からレジスタ${arg1}に読み込み (値: ${value})`;
        break;
        
      case 'WRITE':
        const writeAddr = parseInt(arg2);
        const writeValue = registers[arg1];
        setMemory(prev => {
          const newMem = [...prev];
          newMem[writeAddr] = writeValue;
          return newMem;
        });
        message = `レジスタ${arg1}の値をメモリ[${writeAddr}]に書き込み (値: ${writeValue})`;
        break;
        
      case 'ADD':
        const sum = registers[arg1] + registers[arg2];
        setRegisters(prev => ({ ...prev, [arg1]: sum }));
        message = `${arg1} = ${arg1} + ${arg2} (結果: ${sum})`;
        break;
        
      case 'STOP':
        message = 'プログラム終了';
        return false;
        
      default:
        message = '不明な命令';
        return false;
    }
    
    setLog(prev => [...prev, `PC=${programCounter}: ${cmd} ${arg1} ${arg2} → ${message}`]);
    return true;
  };

  const step = () => {
    if (programCounter >= program.length) {
      setIsRunning(false);
      setLog(prev => [...prev, 'プログラムの終端に到達しました']);
      return;
    }

    const currentInstruction = program[programCounter];
    const instructionText = `${currentInstruction.cmd} ${currentInstruction.arg1} ${currentInstruction.arg2}`.trim();
    setInstructionRegister(instructionText);
    
    const shouldContinue = executeInstruction(currentInstruction);
    
    if (shouldContinue) {
      setProgramCounter(prev => prev + 1);
    } else {
      setIsRunning(false);
    }
  };

  const run = () => {
    setIsRunning(true);
    let pc = programCounter;
    let currentRegisters = { ...registers };
    let currentMemory = [...memory];
    
    const executeStep = () => {
      if (pc >= program.length) {
        setIsRunning(false);
        return;
      }
      
      const currentInstruction = program[pc];
      const instructionText = `${currentInstruction.cmd} ${currentInstruction.arg1} ${currentInstruction.arg2}`.trim();
      
      setInstructionRegister(instructionText);
      setProgramCounter(pc);
      
      let message = '';
      let shouldContinue = true;
      
      switch(currentInstruction.cmd) {
        case 'READ':
          const addr = parseInt(currentInstruction.arg2);
          const value = currentMemory[addr];
          currentRegisters[currentInstruction.arg1] = value;
          setRegisters({ ...currentRegisters });
          message = `メモリ[${addr}]からレジスタ${currentInstruction.arg1}に読み込み (値: ${value})`;
          break;
          
        case 'WRITE':
          const writeAddr = parseInt(currentInstruction.arg2);
          const writeValue = currentRegisters[currentInstruction.arg1];
          currentMemory[writeAddr] = writeValue;
          setMemory([...currentMemory]);
          message = `レジスタ${currentInstruction.arg1}の値をメモリ[${writeAddr}]に書き込み (値: ${writeValue})`;
          break;
          
        case 'ADD':
          const sum = currentRegisters[currentInstruction.arg1] + currentRegisters[currentInstruction.arg2];
          currentRegisters[currentInstruction.arg1] = sum;
          setRegisters({ ...currentRegisters });
          message = `${currentInstruction.arg1} = ${currentInstruction.arg1} + ${currentInstruction.arg2} (結果: ${sum})`;
          break;
          
        case 'STOP':
          message = 'プログラム終了';
          shouldContinue = false;
          break;
      }
      
      setLog(prev => [...prev, `PC=${pc}: ${instructionText} → ${message}`]);
      
      if (!shouldContinue) {
        setIsRunning(false);
        return;
      }
      
      pc++;
      setTimeout(executeStep, 1000);
    };
    
    executeStep();
  };

  const reset = () => {
    setRegisters({ A: 0, B: 0 });
    setMemory(Array(10).fill(0));
    setProgramCounter(0);
    setInstructionRegister('');
    setIsRunning(false);
    setLog([]);
  };

  const updateProgramLine = (index, field, value) => {
    const newProgram = [...program];
    newProgram[index] = { ...newProgram[index], [field]: value };
    setProgram(newProgram);
  };

  const addProgramLine = () => {
    setProgram([...program, { cmd: 'READ', arg1: 'A', arg2: '0' }]);
  };

  const deleteProgramLine = (index) => {
    if (program.length > 1) {
      setProgram(program.filter((_, i) => i !== index));
    }
  };

  const updateMemory = (index, value) => {
    const newMemory = [...memory];
    newMemory[index] = parseInt(value) || 0;
    setMemory(newMemory);
  };

  const getArg1Options = (cmd) => {
    switch(cmd) {
      case 'READ':
      case 'WRITE':
      case 'ADD':
        return ['A', 'B'];
      default:
        return [''];
    }
  };

  const getArg2Options = (cmd) => {
    switch(cmd) {
      case 'READ':
      case 'WRITE':
        return ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'];
      case 'ADD':
        return ['A', 'B'];
      default:
        return [''];
    }
  };

  return (
    <div className="min-h-screen bg-gray-50 p-8">
      <div className="max-w-6xl mx-auto">
        <div className="bg-white rounded border border-gray-300 p-6 mb-6">
          <div className="flex items-center gap-3 mb-4">
            <BookOpen className="w-8 h-8 text-gray-700" />
            <h1 className="text-3xl font-bold text-gray-800">CPU動作シミュレーター</h1>
          </div>
          <p className="text-gray-600">CPUの基本的な動作(フェッチ→デコード→実行)を体験できます</p>
        </div>

        <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
          {/* プログラム入力 */}
          <div className="bg-white rounded border border-gray-300 p-6">
            <h2 className="text-xl font-bold text-gray-800 mb-4">プログラム</h2>
            <div className="space-y-2 mb-4">
              {program.map((line, i) => (
                <div key={i} className="flex gap-2 items-center">
                  <span className={`w-8 text-center font-mono text-sm ${programCounter === i ? 'bg-gray-800 text-white font-bold' : 'bg-gray-200'} rounded px-2 py-1`}>
                    {i}
                  </span>
                  
                  <select
                    value={line.cmd}
                    onChange={(e) => updateProgramLine(i, 'cmd', e.target.value)}
                    className="border border-gray-300 rounded px-2 py-1 text-sm font-mono"
                    disabled={isRunning}
                  >
                    <option value="READ">READ</option>
                    <option value="WRITE">WRITE</option>
                    <option value="ADD">ADD</option>
                    <option value="STOP">STOP</option>
                  </select>
                  
                  {line.cmd !== 'STOP' && (
                    <>
                      <select
                        value={line.arg1}
                        onChange={(e) => updateProgramLine(i, 'arg1', e.target.value)}
                        className="border border-gray-300 rounded px-2 py-1 text-sm font-mono"
                        disabled={isRunning}
                      >
                        {getArg1Options(line.cmd).map(opt => (
                          <option key={opt} value={opt}>{opt}</option>
                        ))}
                      </select>
                      
                      <select
                        value={line.arg2}
                        onChange={(e) => updateProgramLine(i, 'arg2', e.target.value)}
                        className="border border-gray-300 rounded px-2 py-1 text-sm font-mono"
                        disabled={isRunning}
                      >
                        {getArg2Options(line.cmd).map(opt => (
                          <option key={opt} value={opt}>{opt}</option>
                        ))}
                      </select>
                    </>
                  )}
                  
                  <button
                    onClick={() => deleteProgramLine(i)}
                    disabled={isRunning || program.length <= 1}
                    className="ml-auto p-1 text-gray-600 hover:text-red-600 disabled:text-gray-300"
                  >
                    <Trash2 className="w-4 h-4" />
                  </button>
                </div>
              ))}
            </div>
            
            <button
              onClick={addProgramLine}
              disabled={isRunning}
              className="flex items-center gap-2 text-sm text-gray-600 hover:text-gray-800 disabled:text-gray-300 mb-4"
            >
              <Plus className="w-4 h-4" />
              命令を追加
            </button>
            
            <div className="mt-4 p-4 bg-gray-100 rounded border border-gray-300">
              <h3 className="font-semibold text-sm text-gray-800 mb-2">使える命令</h3>
              <div className="text-xs text-gray-700 space-y-1">
                <div><code className="bg-white px-1">READ レジスタ アドレス</code> - メモリからレジスタに読み込む</div>
                <div><code className="bg-white px-1">WRITE レジスタ アドレス</code> - レジスタの値をメモリに書き込む</div>
                <div><code className="bg-white px-1">ADD レジスタ1 レジスタ2</code> - レジスタ1にレジスタ2を加算</div>
                <div><code className="bg-white px-1">STOP</code> - プログラム終了</div>
              </div>
            </div>

            <div className="flex gap-2 mt-4">
              <button
                onClick={run}
                disabled={isRunning}
                className="flex items-center gap-2 bg-gray-800 text-white px-4 py-2 rounded hover:bg-gray-700 disabled:bg-gray-400"
              >
                <Play className="w-4 h-4" />
                実行
              </button>
              <button
                onClick={step}
                disabled={isRunning || programCounter >= program.length}
                className="flex items-center gap-2 bg-gray-600 text-white px-4 py-2 rounded hover:bg-gray-500 disabled:bg-gray-400"
              >
                <StepForward className="w-4 h-4" />
                1ステップ
              </button>
              <button
                onClick={reset}
                className="flex items-center gap-2 bg-gray-500 text-white px-4 py-2 rounded hover:bg-gray-400"
              >
                <RotateCcw className="w-4 h-4" />
                リセット
              </button>
            </div>
          </div>

          {/* CPU状態 */}
          <div className="space-y-6">
            {/* レジスタ */}
            <div className="bg-white rounded border border-gray-300 p-6">
              <h2 className="text-xl font-bold text-gray-800 mb-4">レジスタ(高速記憶領域)</h2>
              <div className="grid grid-cols-2 gap-4">
                {Object.entries(registers).map(([name, value]) => (
                  <div key={name} className="bg-gray-100 border border-gray-300 p-4 rounded text-center">
                    <div className="text-sm font-semibold text-gray-700">{name}</div>
                    <div className="text-3xl font-bold text-gray-900">{value}</div>
                  </div>
                ))}
              </div>
            </div>

            {/* 制御装置 */}
            <div className="bg-white rounded border border-gray-300 p-6">
              <h2 className="text-xl font-bold text-gray-800 mb-4">制御装置</h2>
              <div className="space-y-3">
                <div className="bg-gray-100 border border-gray-300 p-3 rounded">
                  <div className="text-sm font-semibold text-gray-700">プログラムカウンタ (PC)</div>
                  <div className="text-2xl font-bold text-gray-900">{programCounter}</div>
                </div>
                <div className="bg-gray-100 border border-gray-300 p-3 rounded">
                  <div className="text-sm font-semibold text-gray-700">命令レジスタ (IR)</div>
                  <div className="text-sm font-mono text-gray-900">{instructionRegister || '---'}</div>
                </div>
              </div>
            </div>

            {/* メモリ */}
            <div className="bg-white rounded border border-gray-300 p-6">
              <h2 className="text-xl font-bold text-gray-800 mb-4">メモリ(主記憶装置)</h2>
              <div className="grid grid-cols-5 gap-2">
                {memory.map((value, i) => (
                  <div key={i} className="bg-gray-50 border border-gray-300 p-2 rounded">
                    <div className="text-xs font-semibold text-gray-600 text-center mb-1">[{i}]</div>
                    <input
                      type="number"
                      value={value}
                      onChange={(e) => updateMemory(i, e.target.value)}
                      className="w-full text-center text-sm font-bold text-gray-900 border border-gray-300 rounded px-1"
                      disabled={isRunning}
                    />
                  </div>
                ))}
              </div>
            </div>
          </div>
        </div>

        {/* 実行ログ */}
        <div className="bg-white rounded border border-gray-300 p-6 mt-6">
          <h2 className="text-xl font-bold text-gray-800 mb-4">実行ログ</h2>
          <div className="bg-gray-50 border border-gray-300 p-4 rounded font-mono text-sm h-48 overflow-y-auto">
            {log.length === 0 ? (
              <div className="text-gray-400">実行するとログが表示されます</div>
            ) : (
              log.map((entry, i) => (
                <div key={i} className="text-gray-800 mb-1">{entry}</div>
              ))
            )}
          </div>
        </div>
      </div>
    </div>
  );
};

export default CPUSimulator;
