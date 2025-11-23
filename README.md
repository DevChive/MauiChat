# MauiChat

Chat page smaple built with .NET MAUI

## Article Preservation

Learn how to create a simple chat page UI that could be used for real-time message exchange, with support for images.

TLDR: Click here to check a fully working sample with support for Android, iOS and WinUI.

### Objective

Create a chat like interaction, where we can send and receive images and text. Here are some ideas of what it could look like:

<img width="720" height="700" alt="image" src="https://github.com/user-attachments/assets/f89473fd-af28-4b08-a219-09514e71c718" />

<img width="720" height="762" alt="image" src="https://github.com/user-attachments/assets/760e8eb7-f0fb-4073-a507-463b785601ce" />

### Approach used and why I used it

I tried reducing the external dependencies to a minimum, so I used the native controls offered out-of-the-box by .NET MAUI. I did however use the community toolkit nugets to be able to simplify the code and zoft.MauiExtensions.Core nuget package for some useful extensions.
The initial project was created using the MAUI App Accelerator extension by Matt Lacey

<img width="720" height="151" alt="image" src="https://github.com/user-attachments/assets/2e802f21-b2fe-489d-ac6c-2879b09f942c" />

Nugets used in the project (MauiVersion => 8.0.80)
In a first approach I though about using the CollectionView with the grouping feature but as it turned out, this approach works very well for Android, but has known UI rendering issues on iOS. To workaround this problem, I choose to use a single list of items, that contain 2 types of objects (MessageGroupand MessageItem)

```cs
public class MessageGroup(string name)
{
    public string Name { get; } = name;
}

public class MessageItem
{
    public string? Body { get; set; }

    public DateTime Created { get; set; }

    public bool IsMyMessage { get; set; }

    public List<MediaItem> Attachments { get; set; } = [];
}
```

### Implementation

Start by creating a new project (I used the MAUI App Accelerator extension by Matt Lacey).

Configuration for new project

Initial project structure

<img width="468" height="867" alt="image" src="https://github.com/user-attachments/assets/f07bcbdf-b8e9-4758-86aa-b502fff8ba22" />

<img width="443" height="1011" alt="image" src="https://github.com/user-attachments/assets/0579888d-e23e-4baf-9bf5-101a2dc3b88b" />


I’ll skip the formalities of adding the backbone code and keep focused on the UI of the chat itself. You can take a look at the different components added, in the the github repo here. Here’s an overview of what was added:

— Converters to help format the time.
— Models that represent a message, a message group and a media item (image).
— Services that provides a mocked logic for sending and receiving messages.
— ViewModels to support the chat page functionality.
— Views that contain all the UI implementation.

### Final project structure

Now let’s talk about the UI.
The main piece will be the ChatPage. This page will have a CollectionView to show the messages. Bellow that list there will be an area for the controls to write a message, attach images and send the message. Finally at the bottom, there will be an area to show the images that are attached to the message that will be sent.

<img width="720" height="700" alt="image" src="https://github.com/user-attachments/assets/f4f2ca21-eaa1-4217-b5f4-478a4c5aa82d" />

### Messages List

The CollectionView configuration will be quite simple. It will have a list of messages, bound to the ItemsSource and an ItemTemplate bound to a `MessageTemplateSelector`.

```xaml
<!-- Messages List -->
<CollectionView x:Name="MessagesList" 
                ItemsSource="{Binding GroupedMessages}"
                ItemTemplate="{StaticResource MessageTemplateSelector}"
                ItemsUpdatingScrollMode="KeepLastItemInView"
                Grid.Row="0">
    <CollectionView.ItemsLayout>
        <LinearItemsLayout Orientation="Vertical"
                           ItemSpacing="5" />
    </CollectionView.ItemsLayout>
</CollectionView>
```

The `MessageTemplateSelector` will have 3 templates to choose from: `MessageGroupTemplate`, `MessageSentTemplate` and `MessageReceivedTemplate`.

```xaml
<DataTemplate x:Key="MessageGroupHeaderTemplate">
    <v:MessageGroupHeaderTemplate/>
</DataTemplate>

<DataTemplate x:Key="MessageSentTemplate">
    <v:MessageSentTemplate/>
</DataTemplate>

<DataTemplate x:Key="MessageReceivedTemplate">
    <v:MessageReceivedTemplate/>
</DataTemplate>

<v:MessageTemplateSelector x:Key="MessageTemplateSelector"
                           GroupHeader="{StaticResource MessageGroupHeaderTemplate}"
                           MessageSent="{StaticResource MessageSentTemplate}"
                           MessageReceived="{StaticResource MessageReceivedTemplate}" />
```

```cs
public class MessageTemplateSelector : DataTemplateSelector
{
    public DataTemplate GroupHeader { get; set; } = null!;

    public DataTemplate MessageSent { get; set; } = null!;
    
    public DataTemplate MessageReceived { get; set; } = null!;


    protected override DataTemplate OnSelectTemplate(object item, BindableObject container)
    {
        if (item is MessageGroup)
        {
            return GroupHeader;
        }

        return ((MessageItem)item).IsMyMessage ? MessageSent : MessageReceived;
    }
}
```

```xaml
<!-- MessageGroupHeaderTemplate -->
<Grid xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
      xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
      x:Class="MauiChat.Views.MessageGroupHeaderTemplate"
      xmlns:m="clr-namespace:MauiChat.Models"
      x:DataType="m:MessageGroup">

    <Grid.Resources>

        <Style x:Key="GroupBubbleStyle" TargetType="Border">
            <Setter Property="BackgroundColor" Value="#b7babb" />
            <Setter Property="StrokeShape" Value="RoundRectangle 15" />
            <Setter Property="Padding" Value="5" />
            <Setter Property="Margin" Value="5" />
            <Setter Property="HorizontalOptions" Value="Center" />
            <Setter Property="VerticalOptions" Value="Center" />
        </Style>
        
    </Grid.Resources>

    <Border Style="{StaticResource GroupBubbleStyle}">
        <Label Text="{Binding Name}"/>
    </Border>

</Grid>
<!-- MessageReceivedTemplate -->
<Grid xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
      xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
      x:Class="MauiChat.Views.MessageReceivedTemplate"
      xmlns:converters="clr-namespace:MauiChat.Converters"
      xmlns:m="clr-namespace:MauiChat.Models"
      x:DataType="m:MessageItem"
      ColumnDefinitions="10,.8*,.2*">
 
    <Grid.Resources>
 
        <converters:MessageDateConverter x:Key="MessageDateConverter" />
        
        <Color x:Key="PeerMessageBackgroundColor">#DDDDDD</Color>
        <Color x:Key="MessagesTextColor">#DE000000</Color>
        
    </Grid.Resources>
 
    <Grid Grid.Column="0"
          Margin="0,0,0,5"
          VerticalOptions="End">
        
        <Polygon Points="0,10 10,0 10,20"
                 StrokeThickness="0"
                 Fill="{StaticResource PeerMessageBackgroundColor}" />
    </Grid>
 
    <Border Grid.Column="1"
            StrokeShape="RoundRectangle 5"
            StrokeThickness="0"
            BackgroundColor="{StaticResource PeerMessageBackgroundColor}">
        
        <Grid RowDefinitions="Auto,Auto,Auto"
              ColumnDefinitions="*,Auto"
              Padding="10,5,5,5">
 
            <Label Text="Another person"
                   TextColor="{StaticResource MessagesTextColor}"
                   Grid.Row="0"
                   FontAttributes="Bold" />
 
            <Grid Grid.Row="1"
                  RowDefinitions="Auto,Auto"
                  RowSpacing="6">
                
                <Label Text="{Binding Body}"
                       TextColor="{DynamicResource MessagesTextColor}"
                       Grid.Row="0"/>
 
                <FlexLayout BindableLayout.ItemsSource="{Binding Attachments}"
                            Grid.Row="1"
                            Style="{StaticResource FlexLayoutImageContainerStyle}">
                    <BindableLayout.ItemTemplate>
                        <DataTemplate x:DataType="m:MediaItem">
                            <Image Source="{Binding Source}" Style="{StaticResource ImageSmallStyle}" />
                        </DataTemplate>
                    </BindableLayout.ItemTemplate>
                </FlexLayout>
            </Grid>
 
            <Label Grid.Row="2"
                   Grid.Column="1"
                   Text="{Binding Created, Converter={StaticResource MessageDateConverter}}"
                   TextColor="{DynamicResource MessagesTextColor}"/>
        </Grid>
    </Border>
 
</Grid>
<!-- MessageSentTemplate -->
<Grid xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
      xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
      x:Class="MauiChat.Views.MessageSentTemplate"
      xmlns:converters="clr-namespace:MauiChat.Converters"
      xmlns:m="clr-namespace:MauiChat.Models"
      x:DataType="m:MessageItem"
      ColumnDefinitions=".2*,.8*,15">
 
    <Grid.Resources>
 
        <converters:MessageDateConverter x:Key="MessageDateConverter" />
 
        <Color x:Key="MyMessageBackgroundColor">#5C809D</Color>
        <Color x:Key="MessagesTextColor">#FFFFFF</Color>
        
    </Grid.Resources>
    
    <Border Grid.Column="1"
            StrokeShape="RoundRectangle 5"
            StrokeThickness="0"
            BackgroundColor="{StaticResource MyMessageBackgroundColor}">
        
        <Grid RowDefinitions="Auto,Auto,Auto"
              ColumnDefinitions="*,Auto"
              Padding="10,5,5,5">
 
            <Label Text="Myself"
                   TextColor="{StaticResource MessagesTextColor}"
                   Grid.Row="0"
                   FontAttributes="Bold"/>
 
            <Grid Grid.Row="1"
                  RowDefinitions="Auto,Auto"
                  RowSpacing="6">
                <Label Text="{Binding Body}"
                       TextColor="{StaticResource MessagesTextColor}"
                       Grid.Row="0"/>
 
                <FlexLayout BindableLayout.ItemsSource="{Binding Attachments}"
                            Grid.Row="1"
                            Style="{StaticResource FlexLayoutImageContainerStyle}">
                    <BindableLayout.ItemTemplate>
                        <DataTemplate x:DataType="m:MediaItem">
                            <Image Source="{Binding Source}" Style="{StaticResource ImageSmallStyle}" />
                        </DataTemplate>
                    </BindableLayout.ItemTemplate>
                </FlexLayout>
            </Grid>
 
            <Label Grid.Row="2"
                   Grid.Column="1"
                   Text="{Binding Created, Converter={StaticResource MessageDateConverter}}"
                   TextColor="{StaticResource MessagesTextColor}" />
        </Grid>
    </Border>
 
    <Grid Grid.Column="2"
          Margin="0,0,0,5"
          VerticalOptions="End">
        <Polygon Points="0,0 0,20 10,10"
                 StrokeThickness="0"
                 Fill="{StaticResource MyMessageBackgroundColor}" />
    </Grid>
    
</Grid>
```


### Controls Area

This area provides the interaction controls to write a text message and to add attachments.

```xaml
<!-- Message Input -->
<Grid Grid.Row="1"
      ColumnDefinitions="*,Auto"
      IsEnabled="{Binding IsSendingMessage, Converter={StaticResource InvertedBoolConverter}}">

    <Border Grid.Column="0"
            StrokeShape="RoundRectangle 20"
            Margin="6">
        <Grid ColumnDefinitions="*,Auto,Auto"
              ColumnSpacing="6"
              Margin="6,0,6,0">

            <Editor Grid.Column="0"
                    Placeholder="Write your message..."
                    Text="{Binding MessageBody, Mode=TwoWay}"
                    AutoSize="TextChanges"
                    MaximumHeightRequest="100"
                    VerticalOptions="Center"/>

            <Button Style="{StaticResource ActionButtonStyle}"
                    Command="{Binding TakePhotoCommand}"
                    IsVisible="{Binding IsTyping, Converter={StaticResource InvertedBoolConverter}}"
                    Grid.Column="1">
                <Button.ImageSource>
                    <FontImageSource FontFamily="{x:Static root:IconFontFamily.MaterialIcons}"
                                     Glyph="{x:Static root:IconFontGlyph.Photo_camera}"
                                     Size="{StaticResource FontIconDefaultSize}"
                                     Color="{StaticResource FontIconDefaultColor}"/>
                </Button.ImageSource>
            </Button>

            <Button Style="{StaticResource ActionButtonStyle}"
                    Command="{Binding PickPhotoCommand}"
                    IsVisible="{Binding IsTyping, Converter={StaticResource InvertedBoolConverter}}"
                    Grid.Column="2">
                <Button.ImageSource>
                    <FontImageSource FontFamily="{x:Static root:IconFontFamily.MaterialIcons}"
                                     Glyph="{x:Static root:IconFontGlyph.Image}"
                                     Size="{StaticResource FontIconDefaultSize}"
                                     Color="{StaticResource FontIconDefaultColor}"/>
                </Button.ImageSource>
            </Button>
        </Grid>
    </Border>

    <Button Style="{StaticResource SendButtonStyle}"
            Command="{Binding SendMessageCommand}"
            IsVisible="{Binding IsSendingMessage, Converter={StaticResource InvertedBoolConverter}}"
            Grid.Column="1">
        <Button.ImageSource>
            <FontImageSource FontFamily="{x:Static root:IconFontFamily.MaterialIcons}"
                             Glyph="{x:Static root:IconFontGlyph.Send}"
                             Size="24"/>
        </Button.ImageSource>
    </Button>

    <ActivityIndicator Style="{StaticResource ActivityIndicatorStyle}"
                       IsVisible="{Binding IsSendingMessage}"
                       IsRunning="True"
                       Grid.Column="1" />
</Grid>
```

### Attachments

This area will show a list of the attached images.

```xaml
<!-- Attachments List -->
<Grid Grid.Row="2" RowDefinitions="Auto"
      IsEnabled="{Binding IsSendingMessage, Converter={StaticResource InvertedBoolConverter}}"
      IsVisible="{Binding MessageAttachments, Converter={StaticResource IsListNotNullOrEmptyConverter}}"
      MaximumHeightRequest="150">
    <ScrollView VerticalOptions="Fill" HorizontalOptions="Fill" HeightRequest="150">
        <FlexLayout BindableLayout.ItemsSource="{Binding MessageAttachments}"
                    Style="{StaticResource FlexLayoutImageContainerStyle}">
            <BindableLayout.ItemTemplate>
                <DataTemplate x:DataType="m:MediaItem">
                    <Image Source="{Binding Source}"
                           Style="{StaticResource ImageStyle}">
                        <Image.GestureRecognizers>
                            <TapGestureRecognizer Command="{Binding Source={Reference self},
                                                            Path=BindingContext.RemoveAttachmentCommand}" 
                                                  CommandParameter="{Binding .}"/>
                        </Image.GestureRecognizers>
                    </Image>
                </DataTemplate>
            </BindableLayout.ItemTemplate>
        </FlexLayout>
    </ScrollView>
    
</Grid>
```

### ScrollToBottom behaviour

As an extra behavior, I also wanted to had the possibility to Scroll-To-Bottom. This interaction should be done via a button that is shown, when we’re not already at the bottom of the messages list.

<img width="720" height="762" alt="image" src="https://github.com/user-attachments/assets/d4fa0b12-437f-43a6-8a17-7503958c2fb1" />

### Scroll-To-Bottom UI sample

The button UI implementation is quite simple. We just show an ImageButton on top of the messages list.

```xaml
<!-- GoToBottom Button -->
<ImageButton Grid.Row="0"
             Style="{StaticResource ScrollToBottomImageButtonStyle}"
             Clicked="ButtonScrollToBottom_Clicked"
             x:Name="ButtonScrollToBottom">
    <ImageButton.Source>
        <FontImageSource FontFamily="{x:Static root:IconFontFamily.MaterialIcons}"
                         Glyph="{x:Static root:IconFontGlyph.Keyboard_double_arrow_down}"
                         Size="{StaticResource FontIconDefaultSize}"/>
    </ImageButton.Source>
</ImageButton>
```

To control the visibility and functionality of the button, we use the CollectionView.Scrolled event to check if the last item of the list is visible and act accordingly. We then use the CollectionView.ScrollTo method to force the list to show a specific item (in this case, the last one)

```cs
private void MessagesList_Scrolled(object? sender, ItemsViewScrolledEventArgs e)
{
    ButtonScrollToBottom.IsVisible = e.LastVisibleItemIndex != ViewModel.GroupedMessages.Count - 1;
}

private void ButtonScrollToBottom_Clicked(object sender, EventArgs e)
{
    MessagesList.ScrollTo(ViewModel.GroupedMessages.Count - 1);
}
```

And that’s it! The chat page is ready to be used!
