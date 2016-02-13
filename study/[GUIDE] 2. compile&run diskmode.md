--------------------------------------------------
# 2. compile&run diskmode
--------------------------------------------------
*git ȭ�� ���� �����ʿ� RAW ������ �о��ּ��� !!*
</br>
**. /usr/bin �� �ֿ켱������ PATH ���� ����(�ſ� �߿�)**
```
export PATH="/usr/bin:$PATH"
```
���� : ./configure .... install ������ error gcc and g++ ���� ������ ���� ��ɾ� �Է��ϸ� ���� �ذᰡ��!!
���� 2 : terminalâ�� ������ export �ɼ��� �ʱ�ȭ �ǹǷ� terminal â�� ���� ������ �ٽ� �����ϵ��� �Ѵ�.
</br>
**1. Compile grub2**
</br>
�� ���ʷ� ������ �ϴ� ���
```
./linguas.sh
./autogen.sh
./configure --disable-efiemu --prefix=/home/����ڰ�����/g2/usr
make
make install
```
�� �ƴ� ���
```
make
make install
```
�� �߰� ����
```
linguas.sh : �ش� ���� �� �°� ����
autogen.sh : �ϵ��� �´� configure ���� ����
configure : make������ ����, --prefix= ������ �Ȱ��� ������ ���(�����쿡�� ��ġ�� ��� ������ ���� ��)
```
</br>
**2. Create Image file**
```
qemu-img create -f raw /home/����ڰ�����/brdisk-img.raw 1G
sudo mkfs.ext2 /home/����ڰ�����/brdisk-img.raw
sudo losetup /dev/loop0 /home/����ڰ�����/brdisk-img.raw
```
�� mount ����
```
 sudo umount /dev/loop0
```
�� �߰� ����
```
[qemu-img]
Explanation : QEMU ��ũ �̹��� ��ƿ��Ƽ
Command : create [-f ��ũ�̹��� ����][-o options] filename [size]
site : http://linux.die.net/man/1/qemu-img

[mkfs(Make File System)]
Explanation : ���Ͻý��� Ÿ������ �����Ͽ� ������ ���Ͻý����� ����, �� ���̵忡���� ext2Ÿ������ ���Ͻý��� ����
Command : mkfs [-V] [-t ���Ͻý���Ÿ��] [���Ͻý��ۿɼ�] ��ġ�̸� [���]

[losetup]
Explanation : loop device�� ����Ʈ, loop device�� ����̽� ����̹��̴�. ��, image file�� ��ġ �Ϲ����� block device�ΰ�ó�� ����� ����Ʈ�� �� �ְ� �ϴ� ����̽� ����̹��̴�.( �̹��� ������ �Ϲ����� block device �� ����ϴ� ���̶�� �����ϸ� ����.)

��ǻ�Ϳ���, ����Ʈ�� ���Ͻý��� ���� ���� �ִ� �Ϸ��� ���ϵ��� ����ڳ� ����� �׷���� �̿��� �� �ֵ��� ����� ��
```
</br>
**3. Mount brdisk-img.raw**
```
mkdir /home/����ڰ�����/boot
sudo mount /dev/loop0 /home/����ڰ�����/boot
```
</br>
**4. Install grub2**
```
sudo ~/g2/usr/sbin/grub-install --force --no-floppy --boot-directory=/home/����ڰ�����/boot /dev/loop0
```
</br>
**5. Run grub2 with qemu **
```
qemu-system-i386 -m 512 -hda /home/����ڰ�����/brdisk-img.raw
```
�� �߰� ����
```
qemu�� ����ȭ ����Ʈ���� ��� �ϳ���. Fabrice Bellard�� ��������� x86 �̿��� ������ ���� ������� ����Ʈ���� ���� ��ü�� ����ӽ� ������ ������ �� �ִٴ� Ư¡�� �ִ�. ���� ��ȯ��(Portable dynamic translation)�� ����Ѵ�.
```