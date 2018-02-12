using Unity;
using Unity.Interception.ContainerIntegration;
using Microsoft.Practices.Unity.InterceptionExtension;
using System.Collections.Generic;
using System;

namespace ConsoleApp1
{
    class A {
        public string Name { get; set; }
        public bool Enabled { get; set; }
        public IEnumerable<A> Items { get; set; }
    }

    

    class Program
    {
        static string[] names = new string[] {
        "c",
        "d"
        };

        static IEnumerable<A> Items = new List<A> {
                new A {
                    Name= "a",
                    Items = new List<A> {
                        new A {
                            Name = "b",
                            Items = new List<A> {
                                new A {
                                    Name = "c",
                                    Items = new List<A> { }
                                },
                                new A {
                                    Name = "d",
                                    Items = new List<A> { }
                                }
                            }
                        }
                    }
                },
                new A {
                    Name = "e",
                    Items = new List<A> {
                        new A {
                            Name = "f",
                            Items = new List<A> {
                                new A {
                                    Name = "g",
                                    Items = new List<A> { }
                                },
                                new A {
                                    Name = "h",
                                    Items = new List<A> { }
                                }
                            }
                        }
                    }
                }
            };

        static void Main(string[] args)
        {
            Decorate(Items.GetEnumerator(), new HashSet<string>(names));
            Console.ReadKey();
        }

        static void Decorate(IEnumerator<A> current, HashSet<string> search) {

            Stack<IEnumerator<A>> stack = new Stack<IEnumerator<A>>();
            Stack<int> totalstack = new Stack<int>(); totalstack.Push(0);
            Stack<int> enabledstack = new Stack<int>(); enabledstack.Push(0);
            stack.Push(current);
            bool preventDefault = false;


            bool drillUp = false;

            do
            {
                var start = stack.Pop();
                var total = totalstack.Pop();
                var enabled = enabledstack.Pop();
                bool drilldown = false;

                while (start.MoveNext() || preventDefault) {
                    //to top
                    if (start.Current == null) {
                        start = stack.Pop();

                        total = totalstack.Pop();
                        enabled = enabledstack.Pop();

                        preventDefault = false;
                        continue;
                    }

                    //operation
                    total++;
                    start.Current.Enabled = (search.Contains(start.Current.Name) || (drilldown));
                    if (start.Current.Enabled) enabled++;

                    //to down
                    if (start.Current != null && start.Current.Items != null) {
                        stack.Push(start);

                        totalstack.Push(total);
                        enabledstack.Push(enabled);
                        total = 0;
                        enabled = 0;

                        var temp = start.Current.Items.GetEnumerator();
                        preventDefault = true;
                        drilldown = start.Current.Enabled;
                        start = temp;
                    }
                }

                //if (iterations == 0 && drillUp && start.Current != null) {
                //    start.Current.Enabled = drillUp;
                //    totalstack = new Stack<int>(); totalstack.Push(0);
                //    enabledstack = new Stack<int>(); enabledstack.Push(0);
                //}

            } while (stack.Count != 0);
        }
    }
}
