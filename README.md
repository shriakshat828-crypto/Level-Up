# Level-Up
this is a platform just like the system in solo leveling,this system will help you to level up in real life
import React, { useState, useEffect } from 'react';
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from "/components/ui/card";
import { Button } from "/components/ui/button";
import { Input } from "/components/ui/input";
import { Label } from "/components/ui/label";

interface Task {
  id: string;
  title: string;
  description: string;
  completed: boolean;
  createdAt: Date;
  completedAt?: Date;
}

interface UserStats {
  level: number;
  exp: number;
  expToNextLevel: number;
  gold: number;
  tasksCompleted: number;
  tasksFailed: number;
  lastUpdated: Date;
}

const SoloLevelingSystem = () => {
  const [tasks, setTasks] = useState<Task[]>([]);
  const [newTaskTitle, setNewTaskTitle] = useState('');
  const [newTaskDescription, setNewTaskDescription] = useState('');
  const [stats, setStats] = useState<UserStats>({
    level: 1,
    exp: 0,
    expToNextLevel: 100,
    gold: 0,
    tasksCompleted: 0,
    tasksFailed: 0,
    lastUpdated: new Date()
  });

  // Load data from localStorage on initial load
  useEffect(() => {
    const savedTasks = localStorage.getItem('soloLevelingTasks');
    const savedStats = localStorage.getItem('soloLevelingStats');
    
    if (savedTasks) {
      setTasks(JSON.parse(savedTasks, (key, value) => {
        if (key === 'createdAt' || key === 'completedAt') return new Date(value);
        return value;
      }));
    }
    
    if (savedStats) {
      setStats(JSON.parse(savedStats, (key, value) => {
        if (key === 'lastUpdated') return new Date(value);
        return value;
      }));
    }
  }, []);

  // Save data to localStorage whenever it changes
  useEffect(() => {
    localStorage.setItem('soloLevelingTasks', JSON.stringify(tasks));
    localStorage.setItem('soloLevelingStats', JSON.stringify(stats));
  }, [tasks, stats]);

  const addTask = () => {
    if (!newTaskTitle.trim()) return;
    
    const newTask: Task = {
      id: Date.now().toString(),
      title: newTaskTitle,
      description: newTaskDescription,
      completed: false,
      createdAt: new Date()
    };
    
    setTasks([...tasks, newTask]);
    setNewTaskTitle('');
    setNewTaskDescription('');
  };

  const completeTask = (taskId: string) => {
    const updatedTasks = tasks.map(task => {
      if (task.id === taskId && !task.completed) {
        // Reward the user
        const newExp = stats.exp + 25;
        let newLevel = stats.level;
        let expToNextLevel = stats.expToNextLevel;
        
        // Check for level up
        if (newExp >= expToNextLevel) {
          newLevel += 1;
          expToNextLevel = newLevel * 100;
        }
        
        setStats({
          ...stats,
          exp: newExp % stats.expToNextLevel,
          level: newLevel,
          expToNextLevel,
          gold: stats.gold + 10,
          tasksCompleted: stats.tasksCompleted + 1,
          lastUpdated: new Date()
        });
        
        return {
          ...task,
          completed: true,
          completedAt: new Date()
        };
      }
      return task;
    });
    
    setTasks(updatedTasks);
  };

  const failTask = (taskId: string) => {
    const updatedTasks = tasks.map(task => {
      if (task.id === taskId && !task.completed) {
        // Penalize the user
        const newExp = Math.max(0, stats.exp - 15);
        setStats({
          ...stats,
          exp: newExp,
          gold: Math.max(0, stats.gold - 5),
          tasksFailed: stats.tasksFailed + 1,
          lastUpdated: new Date()
        });
        
        return {
          ...task,
          completed: false
        };
      }
      return task;
    });
    
    setTasks(updatedTasks);
  };

  const deleteTask = (taskId: string) => {
    setTasks(tasks.filter(task => task.id !== taskId));
  };

  return (
    <div className="min-h-screen bg-background p-4 md:p-8">
      <div className="max-w-6xl mx-auto">
        <header className="mb-8 text-center">
          <h1 className="text-4xl font-bold text-primary mb-2">Solo Leveling Task System</h1>
          <p className="text-muted-foreground">Complete your daily tasks to level up. Fail and face the consequences.</p>
        </header>
        
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6 mb-8">
          {/* User Stats Card */}
          <Card className="lg:col-span-1">
            <CardHeader>
              <CardTitle>Hunter Stats</CardTitle>
              <CardDescription>Your current progression</CardDescription>
            </CardHeader>
            <CardContent>
              <div className="space-y-4">
                <div>
                  <Label className="text-muted-foreground">Level</Label>
                  <p className="text-3xl font-bold text-primary">{stats.level}</p>
                </div>
                
                <div>
                  <Label className="text-muted-foreground">Experience</Label>
                  <div className="w-full bg-secondary rounded-full h-4">
                    <div 
                      className="bg-primary h-4 rounded-full" 
                      style={{ width: `${(stats.exp / stats.expToNextLevel) * 100}%` }}
                    ></div>
                  </div>
                  <p className="text-sm text-muted-foreground mt-1">
                    {stats.exp} / {stats.expToNextLevel} EXP
                  </p>
                </div>
                
                <div>
                  <Label className="text-muted-foreground">Gold</Label>
                  <p className="text-2xl font-bold text-yellow-500">{stats.gold} G</p>
                </div>
                
                <div className="grid grid-cols-2 gap-4">
                  <div>
                    <Label className="text-muted-foreground">Completed</Label>
                    <p className="text-xl font-bold text-green-500">{stats.tasksCompleted}</p>
                  </div>
                  
                  <div>
                    <Label className="text-muted-foreground">Failed</Label>
                    <p className="text-xl font-bold text-destructive">{stats.tasksFailed}</p>
                  </div>
                </div>
              </div>
            </CardContent>
          </Card>
          
          {/* Add Task Form */}
          <Card className="lg:col-span-2">
            <CardHeader>
              <CardTitle>Add New Quest</CardTitle>
              <CardDescription>What task must you complete today?</CardDescription>
            </CardHeader>
            <CardContent>
              <div className="space-y-4">
                <div className="space-y-2">
                  <Label htmlFor="task-title">Quest Title</Label>
                  <Input
                    id="task-title"
                    placeholder="Enter task title"
                    value={newTaskTitle}
                    onChange={(e) => setNewTaskTitle(e.target.value)}
                  />
                </div>
                
                <div className="space-y-2">
                  <Label htmlFor="task-description">Quest Description (Optional)</Label>
                  <Input
                    id="task-description"
                    placeholder="Enter task description"
                    value={newTaskDescription}
                    onChange={(e) => setNewTaskDescription(e.target.value)}
                  />
                </div>
                
                <Button onClick={addTask} className="w-full">
                  Add Quest
                </Button>
              </div>
            </CardContent>
          </Card>
        </div>
        
        {/* Task List */}
        <Card>
          <CardHeader>
            <CardTitle>Current Quests</CardTitle>
            <CardDescription>Your active and completed tasks</CardDescription>
          </CardHeader>
          <CardContent>
            {tasks.length === 0 ? (
              <div className="text-center py-8 text-muted-foreground">
                <p>No quests available. Add a new quest to begin your journey.</p>
              </div>
            ) : (
              <div className="space-y-4">
                {tasks.map((task) => (
                  <Card key={task.id} className={task.completed ? "border-green-200 bg-green-50" : ""}>
                    <CardContent className="p-4">
                      <div className="flex justify-between items-start">
                        <div className="flex-1">
                          <h3 className={`font-semibold ${task.completed ? "line-through text-green-700" : "text-foreground"}`}>
                            {task.title}
                          </h3>
                          {task.description && (
                            <p className="text-sm text-muted-foreground mt-1">{task.description}</p>
                          )}
                          <p className="text-xs text-muted-foreground mt-2">
                            Created: {task.createdAt.toLocaleDateString()}
                            {task.completedAt && ` • Completed: ${task.completedAt.toLocaleDateString()}`}
                          </p>
                        </div>
                        
                        <div className="flex space-x-2">
                          {!task.completed ? (
                            <>
                              <Button 
                                variant="default" 
                                size="sm"
                                onClick={() => completeTask(task.id)}
                              >
                                Complete
                              </Button>
                              <Button 
                                variant="destructive" 
                                size="sm"
                                onClick={() => failTask(task.id)}
                              >
                                Fail
                              </Button>
                            </>
                          ) : (
                            <Button 
                              variant="outline" 
                              size="sm"
                              onClick={() => deleteTask(task.id)}
                            >
                              Remove
                            </Button>
                          )}
                        </div>
                      </div>
                    </CardContent>
                  </Card>
                ))}
              </div>
            )}
          </CardContent>
        </Card>
      </div>
    </div>
  );
};

export default SoloLevelingSystem;
