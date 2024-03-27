using System;
using System.Collections.Generic;
using System.Data;
using System.Data.SqlTypes;
using System.Dynamic;
using System.IO;
using System.Linq;
using System.Net.Http.Headers;
using System.Reflection;
using System.Runtime.Remoting.Metadata.W3cXsd2001;
using System.Security.Claims;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using System.Xml;
using static System.Net.Mime.MediaTypeNames;

namespace ConsoleApp2
{
    //       Файловые подсказки
    //
    //индекс героя 0
    //индекс противника 1
    //
    //индекс серезбряного мечя 0
    //индекс шлема 1
    //индекс нагрудника 2
    //индекс креста бога 3
    //индекс берсерк амулета 4
    //индекс магического плаща 5
    //
    //индекс арены 0
    //
    //ширина сущностей 5
    //ширина предметов 16
    //ширина арены 18

    internal class Program
    {
        static void Main(string[] args)
        {
            int heroMaxHealth = 100;
            int heroHealth = heroMaxHealth;
            int heroDamage = 40;
            int heroArmor = 0;
            int level = 1;           
            bool openGame = true;

            
            ItemList listItems = new ItemList();
            List<Item> items = listItems.CreatingListOfItems();

            Hero hero = new Hero( heroHealth, heroMaxHealth, heroDamage, heroArmor, level);
            Enemy enemy = new Enemy();

            while (openGame)
            {
                Console.CursorVisible = false;
                enemy.CreatingParameters(hero.Level, hero.Health);
                while (hero.Health > 0 && enemy.Health > 0)
                {
                    DrawInterface(hero, enemy);
                    Console.SetCursorPosition(0, 12);
                    hero.TakeDamage(enemy.Damage);
                    enemy.TakeDamage(hero.Damage);
                    Console.ReadKey();
                    Console.Clear();
                }
                if (hero.Health > 0)
                {
                    Console.SetCursorPosition(0, 0);
                    hero.LevelUp();
                    Console.WriteLine("Win");
                    if (items.Count >= 3)
                    {
                        ChoiceItem(hero, enemy, items);
                    }
                    else
                    {
                        Console.WriteLine("dsda");
                    }
                    hero.Health = hero.MaxHealth;
                }
                Console.Clear();
            }
        }

        static void ChoiceItem(Hero hero, Enemy enemy, List<Item> items)
        {
            Random rand = new Random();

            HashSet<int> randomSuggestedItems = new HashSet<int>();

            int[] suggestedItems = new int[3];


            while (randomSuggestedItems.Count < 3)
            {
                randomSuggestedItems.Add(rand.Next(0, items.Count));
            }

            suggestedItems[0] = randomSuggestedItems.ElementAt(0);
            suggestedItems[1] = randomSuggestedItems.ElementAt(1);
            suggestedItems[2] = randomSuggestedItems.ElementAt(2);

            Console.WriteLine("Выберите награду: \n" + items[suggestedItems[0]].Name + '|' + items[suggestedItems[1]].Name + '|' + items[suggestedItems[2]].Name);
            int choice = Convert.ToInt32(Console.ReadLine());
            switch (choice)
            {
                case 1:
                    items[suggestedItems[0]].PassiveBonus(ref hero, ref enemy);
                    items.RemoveAt(suggestedItems[0]);
                    break;
                case 2:
                    items[suggestedItems[1]].PassiveBonus(ref hero, ref enemy);
                    items.RemoveAt(suggestedItems[1]);
                    break;
                case 3:
                    items[suggestedItems[2]].PassiveBonus(ref hero, ref enemy);
                    items.RemoveAt(suggestedItems[2]);
                    break;
            }


        }

        static void DrawInterface(Hero hero, Enemy enemy)
        {
            int arenaPosX = 1;
            int arenaPosY = 1;
            ArenaSprite arenaSprite = new ArenaSprite("arena.txt", arenaPosX, arenaPosY);
            
            EntitySprite heroSprite = new EntitySprite("entity.txt", 0, arenaSprite.PosX + 3, arenaSprite.PosY + 2);
            EntitySprite enemySprite = new EntitySprite("entity.txt", 1, arenaSprite.PosX + 10, arenaSprite.PosY + 2);

            int barsPosX = arenaSprite.Width + arenaPosX + 1;
            int barsPosY = 1;
            BarsSprites heroHPbars = new BarsSprites(10);

            arenaSprite.DrawInterface();
            heroSprite.DrawInterface();
            enemySprite.DrawInterface();
            heroHPbars.DrawBar(hero.Health, hero.MaxHealth, barsPosX, barsPosY, ConsoleColor.Green, ConsoleColor.Red);
        }
        


    }

    class ItemList
    {
        public List<Item> CreatingListOfItems()
        {
            List<Item> listItems = new List<Item>
            {
                new SilverSword(),
                new LeatherHelmet(),
                new IronChainMail(),
                new BerserkerAmulet(),
                new DvineCross(),
                new MagicCloak()
            };
            return listItems;
        }
    }

    class Item
    {
        public int DamageUp;
        public int MaxHealthUp;
        public int ArmorUp;
        public int DamageDown;
        public int MaxHealthDown;
        public int ArmorDown;
        public string Name;

        public virtual void PassiveBonus(ref Hero hero, ref Enemy enemy) { }
    }

    class SilverSword : Item
    {
        public SilverSword()
        {
            Name = "Серебряный меч";
        }

        public void DeafoultPassiveBonusSilverSword(ref int heroDamage, ref int enemyArmor)
        {
            DamageUp = 10;
            ChangingStats changStat = new ChangingStats();
            changStat.UpingDamage(ref heroDamage, DamageUp);
        }
        public override void PassiveBonus(ref Hero hero, ref Enemy enemy)
        {
            DeafoultPassiveBonusSilverSword(ref hero.Damage, ref enemy.Armor);

        }
    }
    class LeatherHelmet : Item
    {
        public LeatherHelmet()
        {
            Name = "Кожаный шлем";
        }
        public void DeafoultPassiveBonusLeatherHelmet(ref int heroMaxHealth)
        {
            MaxHealthUp = 20;
            ChangingStats changStat = new ChangingStats();
            changStat.UpingHealth(ref heroMaxHealth, MaxHealthUp);
        }
        public override void PassiveBonus(ref Hero hero, ref Enemy enemy)
        {
            DeafoultPassiveBonusLeatherHelmet(ref hero.MaxHealth);
        }
    }

    class IronChainMail : Item
    {
        public IronChainMail()
        {
            Name = "Железная кальчуга";
        }
        public void DeafoultPassiveBonusIronChainMail(ref int heroArmor)
        {
            ArmorUp = 7;
            ChangingStats changStat = new ChangingStats();
            changStat.UpingArmor(ref heroArmor, ArmorUp);
        }

        public override void PassiveBonus(ref Hero hero, ref Enemy enemy)
        {
            DeafoultPassiveBonusIronChainMail(ref hero.Armor);
        }

    }

    class BerserkerAmulet : Item
    {
        public BerserkerAmulet()
        {
            Name = "Амулет берсерка";
        }

        public void DeafoultPassiveBonusBerserkerAmulet(ref int heroArmor, ref int heroMaxHealth)
        {
            MaxHealthUp = heroMaxHealth / 10;
            ArmorDown = 5;
            ChangingStats changStat = new ChangingStats();
            changStat.DowningArmor(ref heroArmor, ArmorDown);
            changStat.UpingHealth(ref heroMaxHealth, MaxHealthUp);
        }
        public override void PassiveBonus(ref Hero hero, ref Enemy enemy)
        {
            DeafoultPassiveBonusBerserkerAmulet(ref hero.Armor, ref hero.MaxHealth);
        }
    }

    class DvineCross : Item
    {
        public DvineCross()
        {
            Name = "Божественный крест";
        }

        public void DeafoultPassiveBonusDvineCross(ref int heroArmor, ref int heroDamage)
        {
            ArmorUp = 12;
            DamageDown = (int)(heroDamage / 12.5f);
            ChangingStats changstat = new ChangingStats();
            changstat.UpingArmor(ref heroArmor, ArmorUp);
            changstat.DowningDamage(ref heroDamage, DamageDown);
        }

        public override void PassiveBonus(ref Hero hero, ref Enemy enemy)
        {
            DeafoultPassiveBonusDvineCross(ref hero.Armor, ref hero.Damage);
        }
    }

    class MagicCloak : Item
    {
        public MagicCloak()
        {
            Name = "Магический плащ";
        }

        public void DeafoultPassiveBonusMagicCloak(ref int heroMaxHealth, ref int enemyDamage)
        {
            MaxHealthDown = heroMaxHealth / 20;
            DamageDown = enemyDamage / 10;
            ChangingStats changstat = new ChangingStats();
            changstat.DowningHealth(ref heroMaxHealth, MaxHealthDown);
            changstat.DowningDamage(ref enemyDamage, DamageDown);
        }

        public override void PassiveBonus(ref Hero hero, ref Enemy enemy)
        {
            DeafoultPassiveBonusMagicCloak(ref hero.MaxHealth, ref enemy.Damage);
        }
    }

    class ChangingStats
    {
        public void UpingDamage(ref int damage, int damageUp)
        {
            damage += damageUp;
        }

        public void UpingHealth(ref int maxHealht, int healthUp)
        {
            maxHealht += healthUp;
        }

        public void UpingArmor(ref int armor, int armorUp)
        {
            armor += armorUp;
        }

        public void DowningDamage(ref int damage, int damageDown)
        {
            damage -= damageDown;
        }

        public void DowningHealth(ref int maxHealht, int healthDown)
        {
            maxHealht -= healthDown;
        }

        public void DowningArmor(ref int armor, int armorDown)
        {
            armor -= armorDown;
        }
    }

    class Interface
    {
        public char[,] Sprite;
        public int PosX;
        public int PosY;
        public int Width;
        public int IndexInFile;
        public Interface(string fileSprites, int width, int indexSprite, int posX, int posY)
        {
            Sprite = SplittingSprites(ReadFile(fileSprites), width)[indexSprite];
            PosX = posX;
            PosY = posY;
            Width = width;
            IndexInFile = indexSprite;
        }
        char[,] ReadFile(string path)
        {
            string[] fileString = File.ReadAllLines(path);

            char[,] fileChar = new char[fileString[0].Length, fileString.Length];

            for (int y = 0; y < fileChar.GetLength(0); y++)
            {
                for (int x = 0; x < fileChar.GetLength(1); x++)
                {
                    fileChar[y, x] = fileString[x][y];
                }
            }
            return fileChar;
        }
        List<char[,]> SplittingSprites(char[,] spritePack, int width)
        {
            List<char[,]> listSprite = new List<char[,]>();

            int range = width;
            int n = 0;
            for (int i = 0; i < spritePack.GetLength(0) / width; i++)
            {
                char[,] sprite = new char[width, spritePack.GetLength(1)];
                for (int y = 0; y < spritePack.GetLength(1); y++)
                {
                    for (int x = range - width; x < range; x++)
                    {
                        sprite[x - n, y] = spritePack[x, y];
                    }
                }
                n += width;
                range += width;
                listSprite.Add(sprite);
            }
            return listSprite;
        }

        public void DrawInterface()
        {
            for (int y = 0; y < Sprite.GetLength(1); y++)
            {
                Console.SetCursorPosition(PosX, PosY + y);
                for (int x = 0; x < Sprite.GetLength(0); x++)
                {
                    Console.Write(Sprite[x, y]);
                }
            }
        }
    }

    class ArenaSprite : Interface
    {
        public ArenaSprite(string fileSprites, int posX, int posY) : base(fileSprites, 18, 0, posX, posY) { }

    }

    class EntitySprite : Interface
    {
        public EntitySprite(string fileSprites, int indexSprite, int posX, int posY) : base(fileSprites, 5, indexSprite, posX, posY) { }
    }

    class BarsSprites
    {
        public List<char> Sprite;

        public BarsSprites(int lengthBar)
        {
            Sprite = new List<char>(lengthBar + 2);
            Sprite.Add('[');
            for (int i = 0; i < lengthBar; i++)
            {
                Sprite.Add('█');
            }
            Sprite.Add(']');
        }

        public void DrawBar(int positivePoints, int maxPoint, int posX, int posY, ConsoleColor colorLeft, ConsoleColor colorRight)
        {
            Console.SetCursorPosition(posX, posY);
            Console.Write(Sprite[0]);
            Console.ForegroundColor = colorLeft;
            for (int i = 0; i < Convert.ToSingle(positivePoints) / maxPoint * 10; i++)
            {
                Console.Write(Sprite[1]);
            }
            Console.ForegroundColor = colorRight;
            for (int j = 0; j < Sprite.Count - (Convert.ToSingle(positivePoints) / maxPoint * 10) - 2; j++)
            {
                Console.Write(Sprite[1]);
            }
            Console.ForegroundColor = ConsoleColor.White;
            Console.Write(Sprite[Sprite.Count - 1] + $" {positivePoints}/{maxPoint}");


        }
    }

    class Hero
    {
        public int Health;
        public int MaxHealth;
        public int Damage;
        public int Armor;
        public int Level;

        public Hero( int health, int maxHealth, int damage, int armor, int level)
        {
            Health = health;
            MaxHealth = maxHealth;
            Damage = damage;
            Armor = armor;
            Level = level;
        }

        public void TakeDamage(int incomingDamage)
        {           
            Health -= incomingDamage * (100 - Armor) / 100;
        }

        public void LevelUp()
        {
            ChangingStats upStat = new ChangingStats();
            Level++;
            int upDamage = Level * 5;
            upStat.UpingDamage(ref Damage, upDamage);
            upStat.UpingHealth(ref MaxHealth, Level * 20);

        }

    }

    class Enemy
    {
        public int Health;
        public int Damage;
        public int Armor;
        public void CreatingParameters(int heroLevel, int heroHealth)
        {
            Random rend = new Random();
            Health = rend.Next(100, 100);//heroHealth - 10, heroHealth + (heroLevel * 10));
            Damage = rend.Next(10, 10);
            Armor = rend.Next(0, 0);
        }
        public void TakeDamage(int incomingDamage)
        {
            Health -= incomingDamage * (100 - Armor) / 100;
        }

    }
}
