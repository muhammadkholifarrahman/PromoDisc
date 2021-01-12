# Muhammad Kholif Arrahman
# 19.11.2799
# S1 IF 04
# Aplikasi Promo Discon 
Aplikasi ini berisi menu makanan dan minuman yang telah disediakan dan juga potongan discon dari promo-promo yang disediakan yang bertujuan untuk memesan makanan ataupun minuman

## Tujuannya
- Pengguna dapat memilih makanan dan minuman yang tersedia pada menu
- Pengguna juga dapat menggunakan promo-promo yang telah disediakan
- Pengguna dapat memasukkan alamat rumahnya
- Pengguna dapat mengetahui harga potongan yang didapat
- Pengguna dapat melihat harga sebelum dikenakan promo

## Cara Menggunakan Aplikasi Ini
 1. Pertama kita Klik tombol tambah pada menu 
 2. lalu akan muncul beberapa menu makananan dan minuman
 3. lalu kita tinggal pilih menu yang ingin dipesan
 4. Kita dapat melihat jumlah biaya yang dikenakan 
 5. Jika ingin menggunakan promo klik tanda tambah yang ada di kolom promo 
 6. lalu pilih promo yang diinginkan
 7. Maka akan terlihat biaya yang dikenakan setelah mendapat promo

## Cara Kerja Aplikasi Ini
Ada beberapa model yang digunakan dalam aplikasi ini
antara lain `Promo.cs` 
```csharp
public class Promo
    {
        public string npromo { get; set; }
        public double diskon { get; set; }

        public double diskonInPercent { get; set; }

        public Promo(string npromo, double diskon = 0, double diskonInPercent = 0)
        {
            this.npromo = npromo;
            this.diskon = diskon;
            this.diskonInPercent = diskonInPercent;
        }

```
`Keranjang.cs` 
```csharp
class Keranjang
    {
        List<Item> itemBelanja;
        List<Promo> itemPromo;
        int capacity = 1;
        Pembayaran pembayaran;
        OnKeranjangChangedListener callback;

        public Keranjang(Pembayaran pembayaran, OnKeranjangChangedListener callback)
        {
            this.pembayaran = pembayaran;
            this.itemBelanja = new List<Item>();
            this.itemPromo = new List<Promo>();
            this.callback = callback;
        }

        public List<Item> getItems()
        {
            return this.itemBelanja;
        }

        public List<Promo> getPromos()
        {
            return this.itemPromo;
        }

        public void addItem(Item item)
        {
            this.itemBelanja.Add(item);
            this.callback.addItemSucceed();
            calculateSubTotal();
        }

        public void addPromo(Promo promo)
        {
            if (capacity == 1)
            {
                this.itemPromo.Add(promo);
                this.callback.addPromoSucceed();
                calculateSubTotal();
            }

            else
            {
                MessageBox.Show("Oopss! Kamu hanya dapat menggunakan salah satu voucher aja", "Ok", MessageBoxButton.OK);
            }
        }

        public void removeItem(Item item)
        {
            this.itemBelanja.Remove(item);
            this.callback.removeItemSucceed();
            capacity = 0;
            calculateSubTotal();
        }

        public void removePromo(Promo promo)
        {
            this.itemPromo.Remove(promo);
            this.callback.removePromoSucceed();
            capacity = 1;
            calculateSubTotal();
        }

        private void calculateSubTotal()
        {
            double subtotal = 0;
            double potongan = 0;
            foreach (Item item in itemBelanja)
            {
                subtotal += item.price;
            }

            foreach (Promo promo in itemPromo)
            {
                if (promo.diskonInPercent != 0)
                {

                    if (promo.diskonInPercent == 30)
                    {
                        if (subtotal >= 100000)
                        {
                            potongan -= 30000;
                        }
                        else
                        {
                            potongan -= subtotal * (promo.diskonInPercent / 100);
                        }
                    }
                    else
                    {
                        potongan -= subtotal * (promo.diskonInPercent / 100);
                    }
                }

                if (promo.diskon != 0)
                {
                    potongan -= promo.diskon;
                }
            }
            pembayaran.updateTotal(subtotal, potongan);

        }
        public interface OnKeranjangChangedListener
        {
            void removeItemSucceed();
            void addItemSucceed();

            void removePromoSucceed();
            void addPromoSucceed();
        }



```
`Item.cs`
```csharp
public class Item
    {
        public string title { get; set; }
        public double price { get; set; }

        public Item(string title, double price)
        {
            this.title = title;
            this.price = price;
        }
    }
``` 
`Pembayaran.cs`
```csharp
class Pembayaran
    {
        private double promo = 0;
        private double balance = 0;


        private OnPembayaranChangedListener paymentCallback;

        public Pembayaran(OnPembayaranChangedListener paymentCallback)
        {
            this.paymentCallback = paymentCallback;
        }

        public void setBalance(double balance)
        {
            this.balance = balance;
        }

        public double getBalance()
        {
            return this.balance;
        }

        public double getPromo()
        {
            return this.promo;
        }

        public void setPromo(double promo)
        {
            this.promo = promo;
        }

        public void updateTotal(double subtotal, double potongan)
        {

            double total = subtotal + potongan;
            this.paymentCallback.onPriceUpdated(subtotal, total, potongan);
        }
    }
``` 
dan model control juga dipakai  `PenawaranController.cs` untuk melist daftar menu yang disediakan 
```csharp
 public void SetOnItemSelectedListener(OnPenawaranChangedListener listener)
        {
            this.listener = listener;
        }

        private void generateContentPenawaran()
        {
            Item drink1 = new Item("Coffe Late", 30000);
            Item drink2 = new Item("Black Tea", 20000);
            Item food1 = new Item("Pizza", 75000);
            Item drink3 = new Item("Milk Shake", 15000);
            Item food2 = new Item("Fried Frice Special", 14000);
            Item food4 = new Item("Watermelon Juice", 25000);
            Item drink4 = new Item("Lemon Squash", 30000);

            controller.addItem(drink1);
            controller.addItem(drink2);
            controller.addItem(food1);
            controller.addItem(drink3);
            controller.addItem(food2);
            controller.addItem(food4);
            controller.addItem(drink4);

            listPenawaran.Items.Refresh();
        }
```
 Lalu pada `MainWindow.xaml.cs` kita menginisiasi object dari `Penawaran.xaml.cs` dan juga `PromoM.xaml.cs`, untuk di masukan dalam sebuah list Keranjang. Dan tampilan Total dari semua belanja akan di tampilkan pada ListBox.
```csharp
public MainWindow()
        {
            InitializeComponent();

            pembayaran = new Pembayaran(this);
            pembayaran.setBalance(500000);
            pembayaran.setPromo(0);

            Keranjang keranjang = new Keranjang(pembayaran, this);

            controller = new MainWindowController(keranjang);

            listBoxPesanan.ItemsSource = controller.getSelectedItems();
            listBoxPakaiPromo.ItemsSource = controller.getSelectedPromos();

            initializeView();
        }

        private void initializeView()
        {
            labelSubtotal.Content = 0;
            labelGrantTotal.Content = 0;
            labelPromoFee.Content = (pembayaran.getPromo() > 0) ? - pembayaran.getPromo() : 0;
        }

        public void onPenawaranSelected(Item item)
        {
            controller.addItem(item);
        }

        private void onButtonAddItemClicked(object sender, RoutedEventArgs e)
        {
            Penawaran penawaranWindow = new Penawaran();
            penawaranWindow.SetOnItemSelectedListener(this);
            penawaranWindow.Show();
        }

        private void listBoxPesanan_ItemClicked(object sender, MouseButtonEventArgs e)
        {
            if (MessageBox.Show("Kamu ingin menghapus item ini?",
                    "Konfirmasi", MessageBoxButton.YesNo) == MessageBoxResult.Yes)
            {
                ListBox listBox = sender as ListBox;
                Item item = listBox.SelectedItem as Item;
                controller.deleteSelectedItem(item);
            }
        }
```

