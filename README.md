# Vishalking

import React, { useState, useEffect, useCallback } from 'react';
import { Play, RotateCcw, ChevronUp, ChevronDown, ChevronLeft, ChevronRight } from 'lucide-react';

const GRID_SIZE = 20;
const CELL_SIZE = 20;
const INITIAL_SNAKE = [{ x: 10, y: 10 }];
const INITIAL_DIRECTION = { x: 1, y: 0 };
const GAME_SPEED = 150;

export default function SnakeGame() {
  const [snake, setSnake] = useState(INITIAL_SNAKE);
  const [direction, setDirection] = useState(INITIAL_DIRECTION);
  const [apple, setApple] = useState({ x: 15, y: 15 });
  const [gameStarted, setGameStarted] = useState(false);
  const [gameOver, setGameOver] = useState(false);
  const [score, setScore] = useState(0);
  const [nextDirection, setNextDirection] = useState(INITIAL_DIRECTION);

  const generateApple = useCallback(() => {
    let newApple;
    do {
      newApple = {
        x: Math.floor(Math.random() * GRID_SIZE),
        y: Math.floor(Math.random() * GRID_SIZE)
      };
    } while (snake.some(segment => segment.x === newApple.x && segment.y === newApple.y));
    return newApple;
  }, [snake]);

  const resetGame = () => {
    setSnake(INITIAL_SNAKE);
    setDirection(INITIAL_DIRECTION);
    setNextDirection(INITIAL_DIRECTION);
    setApple({ x: 15, y: 15 });
    setGameStarted(false);
    setGameOver(false);
    setScore(0);
  };

  const handleDirectionChange = useCallback((newDir) => {
    if (!gameStarted || gameOver) return;
    
    // Prevent reversing direction
    if (newDir.x === -direction.x && newDir.y === -direction.y) return;
    
    setNextDirection(newDir);
  }, [gameStarted, gameOver, direction]);

  useEffect(() => {
    if (!gameStarted || gameOver) return;

    const gameLoop = setInterval(() => {
      setDirection(nextDirection);
      
      setSnake(prevSnake => {
        const head = prevSnake[0];
        const newHead = {
          x: head.x + nextDirection.x,
          y: head.y + nextDirection.y
        };

        // Check wall collision
        if (newHead.x < 0 || newHead.x >= GRID_SIZE || newHead.y < 0 || newHead.y >= GRID_SIZE) {
          setGameOver(true);
          return prevSnake;
        }

        // Check self collision
        if (prevSnake.some(segment => segment.x === newHead.x && segment.y === newHead.y)) {
          setGameOver(true);
          return prevSnake;
        }

        const newSnake = [newHead, ...prevSnake];

        // Check apple collision
        if (newHead.x === apple.x && newHead.y === apple.y) {
          setScore(s => s + 10);
          setApple(generateApple());
        } else {
          newSnake.pop();
        }

        return newSnake;
      });
    }, GAME_SPEED);

    return () => clearInterval(gameLoop);
  }, [gameStarted, gameOver, nextDirection, apple, generateApple]);

  return (
    <div className="flex flex-col items-center justify-center min-h-screen bg-gradient-to-br from-green-900 via-green-800 to-emerald-900 p-4">
      <div className="bg-white rounded-2xl shadow-2xl p-6 max-w-md w-full">
        <h1 className="text-3xl font-bold text-center mb-2 text-green-800">
          Palekar Department Studios
        </h1>
        <p className="text-center text-gray-600 mb-4">Classic Snake Game</p>
        
        <div className="bg-green-100 rounded-lg p-4 mb-4">
          <div className="flex justify-between items-center">
            <span className="text-lg font-semibold text-green-800">Score: {score}</span>
            {gameOver && (
              <span className="text-red-600 font-bold">Game Over!</span>
            )}
          </div>
        </div>

        <div 
          className="relative mx-auto border-4 border-green-700 rounded-lg overflow-hidden"
          style={{ 
            width: GRID_SIZE * CELL_SIZE, 
            height: GRID_SIZE * CELL_SIZE,
            backgroundColor: '#1a4d2e'
          }}
        >
          {/* Snake */}
          {snake.map((segment, i) => (
            <div
              key={i}
              className={`absolute ${i === 0 ? 'bg-yellow-400' : 'bg-green-400'} rounded-sm`}
              style={{
                left: segment.x * CELL_SIZE,
                top: segment.y * CELL_SIZE,
                width: CELL_SIZE - 2,
                height: CELL_SIZE - 2,
                boxShadow: i === 0 ? '0 0 10px rgba(250, 204, 21, 0.8)' : 'none'
              }}
            />
          ))}
          
          {/* Apple */}
          <div
            className="absolute bg-red-500 rounded-full flex items-center justify-center text-white text-xs font-bold"
            style={{
              left: apple.x * CELL_SIZE,
              top: apple.y * CELL_SIZE,
              width: CELL_SIZE - 2,
              height: CELL_SIZE - 2,
              boxShadow: '0 0 15px rgba(239, 68, 68, 0.8)'
            }}
          >
            🍎
          </div>

          {/* Game Over / Start Overlay */}
          {(!gameStarted || gameOver) && (
            <div className="absolute inset-0 bg-black bg-opacity-70 flex items-center justify-center">
              <div className="text-center">
                <p className="text-white text-xl mb-4">
                  {gameOver ? 'Game Over!' : 'Ready to Play?'}
                </p>
                {gameOver && (
                  <p className="text-yellow-400 text-lg mb-4">Final Score: {score}</p>
                )}
              </div>
            </div>
          )}
        </div>

        {/* Controls */}
        <div className="mt-6">
          <div className="grid grid-cols-3 gap-2 max-w-xs mx-auto mb-4">
            <div />
            <button
              onClick={() => handleDirectionChange({ x: 0, y: -1 })}
              className="bg-green-600 hover:bg-green-700 text-white p-4 rounded-lg active:scale-95 transition-transform"
            >
              <ChevronUp className="w-6 h-6 mx-auto" />
            </button>
            <div />
            
            <button
              onClick={() => handleDirectionChange({ x: -1, y: 0 })}
              className="bg-green-600 hover:bg-green-700 text-white p-4 rounded-lg active:scale-95 transition-transform"
            >
              <ChevronLeft className="w-6 h-6 mx-auto" />
            </button>
            <div className="flex items-center justify-center">
              <div className="w-3 h-3 bg-green-400 rounded-full" />
            </div>
            <button
              onClick={() => handleDirectionChange({ x: 1, y: 0 })}
              className="bg-green-600 hover:bg-green-700 text-white p-4 rounded-lg active:scale-95 transition-transform"
            >
              <ChevronRight className="w-6 h-6 mx-auto" />
            </button>
            
            <div />
            <button
              onClick={() => handleDirectionChange({ x: 0, y: 1 })}
              className="bg-green-600 hover:bg-green-700 text-white p-4 rounded-lg active:scale-95 transition-transform"
            >
              <ChevronDown className="w-6 h-6 mx-auto" />
            </button>
            <div />
          </div>

          <div className="flex gap-2 justify-center">
            {!gameStarted || gameOver ? (
              <button
                onClick={() => {
                  if (gameOver) resetGame();
                  setGameStarted(true);
                }}
                className="flex items-center gap-2 bg-green-600 hover:bg-green-700 text-white px-6 py-3 rounded-lg font-semibold transition-colors"
              >
                <Play className="w-5 h-5" />
                {gameOver ? 'Play Again' : 'Start Game'}
              </button>
            ) : (
              <button
                onClick={resetGame}
                className="flex items-center gap-2 bg-red-600 hover:bg-red-700 text-white px-6 py-3 rounded-lg font-semibold transition-colors"
              >
                <RotateCcw className="w-5 h-5" />
                Reset
              </button>
            )}
          </div>
        </div>

        <div className="mt-6 text-center text-sm text-gray-600">
          <p className="font-semibold mb-1">🍎 Collect Tejaswi Apples!</p>
          <p>Use the arrow buttons to control the snake</p>
        </div>
      </div>
    </div>
  );
}
