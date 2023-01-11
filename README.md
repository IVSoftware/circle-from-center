As I now understand from your comment, you require the circle (or line) to remain centered both for size changes of the box itself, but also as the circle expands or contracts based on the mouse wheel. Here is a version that implements basic functionality for both.

[![demo][1]][1]

    public partial class MainForm : Form
    {
        public MainForm()
        {
            InitializeComponent();
            var path = Path.Combine(
                AppDomain.CurrentDomain.BaseDirectory,
                "Images",
                "image.png"
            );
            pictureBox.Image = new Bitmap(Image.FromFile(path));
            pictureBox.SizeMode = PictureBoxSizeMode.StretchImage;
            pictureBox.Paint += onPaintPictureBox;
            pictureBox.Padding = new Padding(10);
            pictureBox.Anchor = (AnchorStyles)0xF;
            pictureBox.SizeChanged += (sender, e) => pictureBox.Invalidate();
            pictureBox.MouseWheel += onMouseWheelPictureBox;
            // Constrain main form to square.
            SizeChanged += (sender, e) => Size = new Size(Height, Height);
        }

        const int WHEEL_DELTA = 120;
        int notches = 0;
        private void onMouseWheelPictureBox(object? sender, MouseEventArgs e)
        {
            notches += e.Delta / WHEEL_DELTA;
            pictureBox.Invalidate();
        }

        private void onPaintPictureBox(object? sender, PaintEventArgs e)
        {
            var scale = 1.0 - (0.2 * notches);
            var x = (int)(pictureBox.Padding.Left * scale);
            var y = (int)(pictureBox.Padding.Top * scale);
            using (var pen = new Pen(Color.Red, 2f))
            {
                e.Graphics.SmoothingMode = System.Drawing.Drawing2D.SmoothingMode.AntiAlias;
                e.Graphics.DrawEllipse(
                    pen,
                    x, 
                    y,
                    width: e.ClipRectangle.Width - (x * 2),
                    height: e.ClipRectangle.Height - (y * 2));
            }
            using (var pen = new Pen(Color.Red, 2f))
            {
                e.Graphics.SmoothingMode = System.Drawing.Drawing2D.SmoothingMode.AntiAlias;
                e.Graphics.DrawLine(
                    pen,
                    x1: (float)pictureBox.Padding.Left,
                    y1: pictureBox.Padding.Top,
                    x2: e.ClipRectangle.Width - pictureBox.Padding.Left,
                    y2: e.ClipRectangle.Height - pictureBox.Padding.Bottom);
            }
        }

  [1]: https://i.stack.imgur.com/qEIFP.jpg