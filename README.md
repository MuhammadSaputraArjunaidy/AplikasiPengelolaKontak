
# Aplikasi Pengelola Kontak



# Deskripsi
aplikasi ini memungkinkan pengguna untuk mengelola daftar kontak, dengan fungsi untuk menambah, mengedit, menghapus, dan mencari kontak berdasarkan nama atau kategori.
# Features

Berikut adalah deskripsi fitur-fitur utama aplikasi ini:

1. Kolom Input:

- Nama: Kolom teks untuk memasukkan nama kontak.
- Nomor Telepon: Kolom teks untuk memasukkan nomor telepon kontak.
- Kategori: Dropdown menu yang memungkinkan pengguna untuk memilih kategori kontak, misalnya "Keluarga."

2. Bagian Pencarian:

- Cari: Kolom input untuk mencari kontak berdasarkan nama atau kategori.
- Cari Kontak: Tombol untuk memulai pencarian berdasarkan nilai yang dimasukkan dalam kolom "Cari".

3. Tombol:

- Tambah: Tombol untuk menambahkan kontak baru ke dalam daftar.
- Edit: Tombol untuk mengedit informasi kontak yang sudah ada.
- Hapus: Tombol untuk menghapus kontak.
- Simpan: Tombol untuk menyimpan informasi kontak yang telah dimasukkan.

4. Tabel Kontak:

- Tabel di bagian bawah menampilkan daftar kontak dengan kolom-kolom:
- Id: ID unik untuk setiap kontak.
- Nama: Nama dari kontak.
- Nomor Telepon: Nomor telepon dari kontak.
- Kategori: Kategori kontak. 
 # Source Code
 ```java
           
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.Statement;
import javax.swing.JOptionPane;
import javax.swing.table.DefaultTableModel;
import java.io.FileWriter;
import java.awt.event.ItemEvent;
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;


public class PengelolaKontakFrame extends javax.swing.JFrame {

    /**
     * Creates new form PengelolaKontakFrame
     */
    public PengelolaKontakFrame() {
        initComponents(); // Inisialisasi komponen GUI
        
        loadContactsToTable(); // Memuat data ke JTable saat aplikasi dijalankan
    }

   

    // Metode untuk menambahkan kontak ke database
    private void addContact(String name, String phone, String category) {
        String sql = "INSERT INTO contacts(name, phone, category) VALUES(?,?,?)";
        try (Connection conn = DriverManager.getConnection("jdbc:sqlite:contacts.db");
                PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setString(1, name);
            pstmt.setString(2, phone);
            pstmt.setString(3, category);
            pstmt.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // Metode untuk memuat kontak dari database ke JTable
    private void loadContactsToTable() {
        String sql = "SELECT * FROM contacts";
        DefaultTableModel model = (DefaultTableModel) contactsTable.getModel();
        model.setRowCount(0); // Bersihkan tabel sebelum mengisi ulang

        try (Connection conn = DriverManager.getConnection("jdbc:sqlite:contacts.db");
                Statement stmt = conn.createStatement();
                ResultSet rs = stmt.executeQuery(sql)) {

            while (rs.next()) {
                Object[] row = {rs.getInt("id"), rs.getString("name"), rs.getString("phone"), rs.getString("category")};
                model.addRow(row);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // Metode untuk memperbarui kontak di database
    private void updateContact(int id, String name, String phone, String category) {
        String sql = "UPDATE contacts SET name = ?, phone = ?, category = ? WHERE id = ?";
        try (Connection conn = DriverManager.getConnection("jdbc:sqlite:contacts.db");
                PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setString(1, name);
            pstmt.setString(2, phone);
            pstmt.setString(3, category);
            pstmt.setInt(4, id);
            pstmt.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // Metode untuk menghapus kontak dari database
    private void deleteContact(int id) {
        String sql = "DELETE FROM contacts WHERE id = ?";
        try (Connection conn = DriverManager.getConnection("jdbc:sqlite:contacts.db");
                PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setInt(1, id);
            pstmt.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private void searchContacts(String query) {
        String sql = "SELECT * FROM contacts WHERE name LIKE ? OR phone LIKE ?";
        DefaultTableModel model = (DefaultTableModel) contactsTable.getModel();
        model.setRowCount(0); // Bersihkan tabel sebelum mengisi ulang

        try (Connection conn = DriverManager.getConnection("jdbc:sqlite:contacts.db");
                PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setString(1, "%" + query + "%");
            pstmt.setString(2, "%" + query + "%");
            ResultSet rs = pstmt.executeQuery();

            while (rs.next()) {
                Object[] row = {rs.getInt("id"), rs.getString("name"), rs.getString("phone"), rs.getString("category")};
                model.addRow(row);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // Metode untuk validasi nomor telepon
    private boolean isValidPhoneNumber(String phone) {
        return phone.matches("\\d+") && phone.length() >= 10;
    }

    // Metode untuk ekspor kontak ke CSV
    private void exportToCSV(String filePath) {
        try (Connection conn = DriverManager.getConnection("jdbc:sqlite:contacts.db");
                Statement stmt = conn.createStatement();
                ResultSet rs = stmt.executeQuery("SELECT name, phone, category FROM contacts");
                FileWriter csvWriter = new FileWriter(filePath)) {

            csvWriter.append("Name,Phone,Category\n");
            while (rs.next()) {
                csvWriter.append(rs.getString("name") + "," + rs.getString("phone") + "," + rs.getString("category") + "\n");
            }
            JOptionPane.showMessageDialog(this, "Data kontak berhasil diekspor ke " + filePath);
        } catch (Exception e) {
            e.printStackTrace();
            JOptionPane.showMessageDialog(this, "Gagal mengekspor data ke CSV: " + e.getMessage());
        }
    }

    // Metode untuk impor kontak dari CSV
    private void importFromCSV(String filePath) {
        String line;
        try (BufferedReader br = new BufferedReader(new FileReader(filePath))) {
            while ((line = br.readLine()) != null) {
                String[] values = line.split(",");
                if (values.length == 3) {
                    String name = values[0].trim();
                    String phone = values[1].trim();
                    String category = values[2].trim();

                    // Validasi nomor telepon sebelum menambahkan
                    if (isValidPhoneNumber(phone)) {
                        addContact(name, phone, category); // Masukkan ke database
                    } else {
                        JOptionPane.showMessageDialog(this, "Data tidak valid di CSV: " + line);
                    }
                }
            }
            JOptionPane.showMessageDialog(this, "Impor CSV selesai.");
            loadContactsToTable(); // Refresh table setelah impor
        } catch (Exception e) {
            e.printStackTrace();
            JOptionPane.showMessageDialog(this, "Gagal mengimpor data dari CSV: " + e.getMessage());
        }
    }

    // ItemListener untuk JComboBox kategori (opsional, dapat dikembangkan lebih lanjut)
    private void categoryComboBoxItemStateChanged(ItemEvent e) {
        if (e.getStateChange() == ItemEvent.SELECTED) {
            String selectedCategory = (String) e.getItem();
            // Tambahkan logika tambahan jika perlu, berdasarkan kategori yang dipilih
        }
    }

    // Variabel untuk menyimpan ID kontak yang dipilih
    private Integer selectedContactId = null;

// Metode untuk memuat data kontak yang dipilih ke field input
    private void loadSelectedContactToFields() {
        int selectedRow = contactsTable.getSelectedRow();
        if (selectedRow != -1) {
            selectedContactId = (Integer) contactsTable.getValueAt(selectedRow, 0);
            String name = (String) contactsTable.getValueAt(selectedRow, 1);
            String phone = (String) contactsTable.getValueAt(selectedRow, 2);
            String category = (String) contactsTable.getValueAt(selectedRow, 3);

            nameTextField.setText(name);
            phoneTextField.setText(phone);
            categoryComboBox.setSelectedItem(category);
        }
    }

    // Metode untuk mengosongkan field input
    private void clearFields() {
        nameTextField.setText("");
        phoneTextField.setText("");
        categoryComboBox.setSelectedIndex(0);
    }

    /**
     * This method is called from within the constructor to initialize the form.
     * WARNING: Do NOT modify this code. The content of this method is always
     * regenerated by the Form Editor.
     */
    @SuppressWarnings("unchecked")
    // <editor-fold defaultstate="collapsed" desc="Generated Code">                          
    private void initComponents() {

        jPanel1 = new javax.swing.JPanel();
        cariButton = new javax.swing.JButton();
        jLabel1 = new javax.swing.JLabel();
        jLabel2 = new javax.swing.JLabel();
        jLabel3 = new javax.swing.JLabel();
        nameTextField = new javax.swing.JTextField();
        phoneTextField = new javax.swing.JTextField();
        categoryComboBox = new javax.swing.JComboBox<>();
        tambahButton = new javax.swing.JButton();
        editButton = new javax.swing.JButton();
        hapusButton = new javax.swing.JButton();
        jScrollPane1 = new javax.swing.JScrollPane();
        contactsTable = new javax.swing.JTable();
        eksporButton = new javax.swing.JButton();
        cariTextField = new javax.swing.JTextField();
        jLabel4 = new javax.swing.JLabel();

        setDefaultCloseOperation(javax.swing.WindowConstants.EXIT_ON_CLOSE);

        cariButton.setText("Cari Kontak");
        cariButton.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                cariButtonActionPerformed(evt);
            }
        });

        jLabel1.setText("Nama : ");

        jLabel2.setText("Nomor Telpon : ");

        jLabel3.setText("Kategori : ");

        categoryComboBox.setModel(new javax.swing.DefaultComboBoxModel<>(new String[] { "Keluarga", "Teman", "Kerja" }));
        categoryComboBox.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                categoryComboBoxActionPerformed(evt);
            }
        });

        tambahButton.setText("Tambah");
        tambahButton.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                tambahButtonActionPerformed(evt);
            }
        });

        editButton.setText("Edit");
        editButton.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                editButtonActionPerformed(evt);
            }
        });

        hapusButton.setText("Hapus");
        hapusButton.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                hapusButtonActionPerformed(evt);
            }
        });

        contactsTable.setModel(new javax.swing.table.DefaultTableModel(
            new Object [][] {
                {null, null, null, null},
                {null, null, null, null},
                {null, null, null, null},
                {null, null, null, null}
            },
            new String [] {
                "Id", "Nama", "Nomor Telpon", "Kategori"
            }
        ));
        jScrollPane1.setViewportView(contactsTable);

        eksporButton.setText("Simpan");
        eksporButton.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                eksporButtonActionPerformed(evt);
            }
        });

        jLabel4.setText("cari :");

        javax.swing.GroupLayout jPanel1Layout = new javax.swing.GroupLayout(jPanel1);
        jPanel1.setLayout(jPanel1Layout);
        jPanel1Layout.setHorizontalGroup(
            jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
            .addGroup(jPanel1Layout.createSequentialGroup()
                .addContainerGap()
                .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                    .addGroup(jPanel1Layout.createSequentialGroup()
                        .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                            .addGroup(jPanel1Layout.createSequentialGroup()
                                .addGap(5, 5, 5)
                                .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                                    .addComponent(jLabel2)
                                    .addComponent(jLabel3)
                                    .addComponent(jLabel1))
                                .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.RELATED)
                                .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                                    .addComponent(nameTextField, javax.swing.GroupLayout.PREFERRED_SIZE, 109, javax.swing.GroupLayout.PREFERRED_SIZE)
                                    .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.TRAILING, false)
                                        .addComponent(categoryComboBox, javax.swing.GroupLayout.Alignment.LEADING, 0, javax.swing.GroupLayout.DEFAULT_SIZE, Short.MAX_VALUE)
                                        .addComponent(phoneTextField, javax.swing.GroupLayout.Alignment.LEADING, javax.swing.GroupLayout.PREFERRED_SIZE, 108, javax.swing.GroupLayout.PREFERRED_SIZE))))
                            .addGroup(javax.swing.GroupLayout.Alignment.TRAILING, jPanel1Layout.createSequentialGroup()
                                .addComponent(tambahButton)
                                .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.RELATED)
                                .addComponent(editButton)
                                .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.RELATED)
                                .addComponent(hapusButton)))
                        .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                            .addGroup(jPanel1Layout.createSequentialGroup()
                                .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.RELATED)
                                .addComponent(eksporButton))
                            .addGroup(javax.swing.GroupLayout.Alignment.TRAILING, jPanel1Layout.createSequentialGroup()
                                .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.RELATED)
                                .addComponent(cariButton, javax.swing.GroupLayout.PREFERRED_SIZE, 97, javax.swing.GroupLayout.PREFERRED_SIZE)
                                .addGap(19, 19, 19))
                            .addGroup(jPanel1Layout.createSequentialGroup()
                                .addGap(116, 116, 116)
                                .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                                    .addComponent(jLabel4)
                                    .addComponent(cariTextField, javax.swing.GroupLayout.PREFERRED_SIZE, 138, javax.swing.GroupLayout.PREFERRED_SIZE)))))
                    .addComponent(jScrollPane1, javax.swing.GroupLayout.PREFERRED_SIZE, 490, javax.swing.GroupLayout.PREFERRED_SIZE))
                .addContainerGap(50, Short.MAX_VALUE))
        );
        jPanel1Layout.setVerticalGroup(
            jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
            .addGroup(jPanel1Layout.createSequentialGroup()
                .addContainerGap()
                .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                    .addGroup(jPanel1Layout.createSequentialGroup()
                        .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.BASELINE)
                            .addComponent(jLabel1)
                            .addComponent(nameTextField, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE))
                        .addGap(20, 20, 20))
                    .addGroup(javax.swing.GroupLayout.Alignment.TRAILING, jPanel1Layout.createSequentialGroup()
                        .addComponent(jLabel4)
                        .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.RELATED)))
                .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.BASELINE)
                    .addComponent(jLabel2)
                    .addComponent(phoneTextField, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE)
                    .addComponent(cariTextField, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE))
                .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.UNRELATED)
                .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.BASELINE)
                    .addComponent(jLabel3)
                    .addComponent(categoryComboBox, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE)
                    .addComponent(cariButton))
                .addGap(25, 25, 25)
                .addGroup(jPanel1Layout.createParallelGroup(javax.swing.GroupLayout.Alignment.BASELINE)
                    .addComponent(editButton)
                    .addComponent(hapusButton)
                    .addComponent(tambahButton)
                    .addComponent(eksporButton))
                .addGap(18, 18, 18)
                .addComponent(jScrollPane1, javax.swing.GroupLayout.PREFERRED_SIZE, 97, javax.swing.GroupLayout.PREFERRED_SIZE)
                .addContainerGap(68, Short.MAX_VALUE))
        );

        javax.swing.GroupLayout layout = new javax.swing.GroupLayout(getContentPane());
        getContentPane().setLayout(layout);
        layout.setHorizontalGroup(
            layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
            .addGroup(layout.createSequentialGroup()
                .addContainerGap()
                .addComponent(jPanel1, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE)
                .addContainerGap(javax.swing.GroupLayout.DEFAULT_SIZE, Short.MAX_VALUE))
        );
        layout.setVerticalGroup(
            layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
            .addGroup(javax.swing.GroupLayout.Alignment.TRAILING, layout.createSequentialGroup()
                .addContainerGap(javax.swing.GroupLayout.DEFAULT_SIZE, Short.MAX_VALUE)
                .addComponent(jPanel1, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE)
                .addContainerGap())
        );

        pack();
    }// </editor-fold>                        

    private void hapusButtonActionPerformed(java.awt.event.ActionEvent evt) {                                            
        int selectedRow = contactsTable.getSelectedRow();
        if (selectedRow != -1) {
            int id = (int) contactsTable.getValueAt(selectedRow, 0);
            deleteContact(id);
            loadContactsToTable(); // Refresh table
        }// TODO add your handling code here:
    }                                           

    private void tambahButtonActionPerformed(java.awt.event.ActionEvent evt) {                                             
        String name = nameTextField.getText();
        String phone = phoneTextField.getText();
        String category = (String) categoryComboBox.getSelectedItem();

        if (!isValidPhoneNumber(phone)) {
            JOptionPane.showMessageDialog(this, "Nomor telepon harus berupa angka dan memiliki panjang minimal 10 karakter.");
            return;
        }

        addContact(name, phone, category);
        loadContactsToTable(); // Refresh table// TODO add your handling code here:
    }                                            

    private void editButtonActionPerformed(java.awt.event.ActionEvent evt) {                                           
        if (selectedContactId != null) {
            String name = nameTextField.getText();
            String phone = phoneTextField.getText();
            String category = (String) categoryComboBox.getSelectedItem();

            if (!isValidPhoneNumber(phone)) {
                JOptionPane.showMessageDialog(this, "Nomor telepon harus berupa angka dan memiliki panjang minimal 10 karakter.");
                return;
            }

            updateContact(selectedContactId, name, phone, category);

            selectedContactId = null;
            clearFields();
            loadContactsToTable(); // Refresh table
        } else {
            JOptionPane.showMessageDialog(this, "Pilih kontak yang ingin diedit.");
        }

        // Tambahkan listener untuk memuat data kontak saat baris tabel diklik
        contactsTable.addMouseListener(new java.awt.event.MouseAdapter() {
            public void mouseClicked(java.awt.event.MouseEvent evt) {
                loadSelectedContactToFields();
            }
        });
    }                                          

    private void cariButtonActionPerformed(java.awt.event.ActionEvent evt) {                                           
        String query = cariTextField.getText();
        searchContacts(query);// TODO add your handling code here:
    }                                          

    private void eksporButtonActionPerformed(java.awt.event.ActionEvent evt) {                                             
        // Dialog pilihan untuk Ekspor atau Impor
        String[] options = {"Ekspor CSV", "Impor CSV", "Batal"};
        int choice = JOptionPane.showOptionDialog(this,
                "Pilih tindakan yang ingin Anda lakukan:",
                "Ekspor / Impor CSV",
                JOptionPane.DEFAULT_OPTION,
                JOptionPane.INFORMATION_MESSAGE,
                null, options, options[0]);

        if (choice == 0) { // Ekspor CSV
            String filePath = "contacts.csv"; // Tentukan path file CSV
            exportToCSV(filePath);
        } else if (choice == 1) { // Impor CSV
            String filePath = "contacts.csv"; // Tentukan path file CSV
            importFromCSV(filePath);
        }
    }                                            

    private void categoryComboBoxActionPerformed(java.awt.event.ActionEvent evt) {                                                 
        // TODO add your handling code here:
    }                                                

    /**
     * @param args the command line arguments
     */
    public static void main(String args[]) {
        java.awt.EventQueue.invokeLater(() -> {
            new PengelolaKontakFrame().setVisible(true);
        });
    }


    // Variables declaration - do not modify                     
    private javax.swing.JButton cariButton;
    private javax.swing.JTextField cariTextField;
    private javax.swing.JComboBox<String> categoryComboBox;
    private javax.swing.JTable contactsTable;
    private javax.swing.JButton editButton;
    private javax.swing.JButton eksporButton;
    private javax.swing.JButton hapusButton;
    private javax.swing.JLabel jLabel1;
    private javax.swing.JLabel jLabel2;
    private javax.swing.JLabel jLabel3;
    private javax.swing.JLabel jLabel4;
    private javax.swing.JPanel jPanel1;
    private javax.swing.JScrollPane jScrollPane1;
    private javax.swing.JTextField nameTextField;
    private javax.swing.JTextField phoneTextField;
    private javax.swing.JButton tambahButton;
    // End of variables declaration                   
}

```




# Authors

- MuhammadSaputraArjunaidy
- 2210010300
- 5B Reg Pagi Banjarmasin
- [@MuhammadSaputraArjunaidy](https://www.github.com/MuhammadSaputraArjunaidy)


#  Screenshot